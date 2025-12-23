# Section 3: Data Engineering & SQL

**Part of:** [Part 5: Specialized Workflows & Domains](README.md)  
**Related:** [Backend Development](02-backend-development.md) | [Core Workflows](../01-fundamentals-core-concepts/03-core-workflows.md)

---

## 📋 Overview

This section demonstrates how GitHub Copilot accelerates data engineering workflows, from SQL query generation and optimization to ETL pipelines, database migrations, and data validation. Whether you're working with PostgreSQL, MySQL, MongoDB, or data warehouses like Snowflake, Copilot understands SQL dialects and data transformation patterns.

**What you'll learn:**
- SQL query generation and optimization
- ETL pipeline development
- Data transformation and cleaning scripts
- Database migration generation
- Query performance analysis
- Schema design assistance
- Data validation patterns
- Working with different SQL dialects

**Time to master:** 40-50 minutes

---

## 🎯 Key Takeaways

1. **SQL generation:** Copilot writes complex queries with joins, aggregations, and window functions
2. **Dialect awareness:** Understands PostgreSQL, MySQL, SQL Server, Oracle syntax differences
3. **ETL automation:** Generate data transformation pipelines in Python, Node.js, or SQL
4. **Migration safety:** Auto-generate migrations with rollback scripts
5. **Query optimization:** Copilot suggests indexes and query rewrites
6. **Data quality:** Generate validation scripts and data quality checks

---

## 🚀 Quick Wins: Data Engineering

### Pattern 1: Complex SQL Query Generation

**Scenario:** Generate analytics queries with joins and aggregations.

```
Chat:

"Write PostgreSQL query for e-commerce analytics:
- Total revenue by product category
- Top 10 customers by lifetime value
- Monthly revenue growth rate
- Average order value trend
Include proper indexes and explain plan"
```

**Result:**

```sql
-- Revenue by product category
-- Optimized with indexes on frequently joined/filtered columns

CREATE INDEX IF NOT EXISTS idx_orders_created_at ON orders(created_at);
CREATE INDEX IF NOT EXISTS idx_order_items_order_id ON order_items(order_id);
CREATE INDEX IF NOT EXISTS idx_order_items_product_id ON order_items(product_id);
CREATE INDEX IF NOT EXISTS idx_products_category_id ON products(category_id);

-- Query 1: Total revenue by product category
SELECT 
    c.name AS category_name,
    COUNT(DISTINCT o.id) AS total_orders,
    SUM(oi.quantity) AS total_items_sold,
    SUM(oi.price * oi.quantity) AS total_revenue,
    AVG(oi.price * oi.quantity) AS avg_order_item_value
FROM 
    categories c
    INNER JOIN products p ON c.id = p.category_id
    INNER JOIN order_items oi ON p.id = oi.product_id
    INNER JOIN orders o ON oi.order_id = o.id
WHERE 
    o.status IN ('DELIVERED', 'SHIPPED')
    AND o.created_at >= CURRENT_DATE - INTERVAL '1 year'
GROUP BY 
    c.id, c.name
ORDER BY 
    total_revenue DESC;

-- Query 2: Top 10 customers by lifetime value
SELECT 
    u.id,
    u.email,
    u.first_name || ' ' || u.last_name AS full_name,
    COUNT(DISTINCT o.id) AS total_orders,
    SUM(o.total) AS lifetime_value,
    AVG(o.total) AS avg_order_value,
    MAX(o.created_at) AS last_order_date,
    EXTRACT(DAY FROM (CURRENT_DATE - MAX(o.created_at))) AS days_since_last_order
FROM 
    users u
    INNER JOIN orders o ON u.id = o.user_id
WHERE 
    o.status IN ('DELIVERED', 'SHIPPED')
GROUP BY 
    u.id, u.email, u.first_name, u.last_name
ORDER BY 
    lifetime_value DESC
LIMIT 10;

-- Query 3: Monthly revenue growth rate
WITH monthly_revenue AS (
    SELECT 
        DATE_TRUNC('month', o.created_at) AS month,
        SUM(o.total) AS revenue
    FROM 
        orders o
    WHERE 
        o.status IN ('DELIVERED', 'SHIPPED')
        AND o.created_at >= CURRENT_DATE - INTERVAL '2 years'
    GROUP BY 
        DATE_TRUNC('month', o.created_at)
),
revenue_with_previous AS (
    SELECT 
        month,
        revenue,
        LAG(revenue) OVER (ORDER BY month) AS previous_month_revenue
    FROM 
        monthly_revenue
)
SELECT 
    month,
    revenue,
    previous_month_revenue,
    CASE 
        WHEN previous_month_revenue IS NULL THEN NULL
        ELSE ROUND(
            ((revenue - previous_month_revenue) / previous_month_revenue * 100)::numeric, 
            2
        )
    END AS growth_rate_percentage
FROM 
    revenue_with_previous
ORDER BY 
    month DESC;

-- Query 4: Average order value trend (7-day moving average)
WITH daily_orders AS (
    SELECT 
        DATE(o.created_at) AS order_date,
        AVG(o.total) AS avg_order_value,
        COUNT(*) AS order_count
    FROM 
        orders o
    WHERE 
        o.status IN ('DELIVERED', 'SHIPPED')
        AND o.created_at >= CURRENT_DATE - INTERVAL '90 days'
    GROUP BY 
        DATE(o.created_at)
)
SELECT 
    order_date,
    avg_order_value,
    order_count,
    AVG(avg_order_value) OVER (
        ORDER BY order_date 
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS moving_avg_7day
FROM 
    daily_orders
ORDER BY 
    order_date DESC;

-- Performance analysis with EXPLAIN
EXPLAIN ANALYZE
SELECT 
    c.name AS category_name,
    SUM(oi.price * oi.quantity) AS total_revenue
FROM 
    categories c
    INNER JOIN products p ON c.id = p.category_id
    INNER JOIN order_items oi ON p.id = oi.product_id
    INNER JOIN orders o ON oi.order_id = o.id
WHERE 
    o.status IN ('DELIVERED', 'SHIPPED')
    AND o.created_at >= CURRENT_DATE - INTERVAL '1 year'
GROUP BY 
    c.id, c.name
ORDER BY 
    total_revenue DESC;
```

---

### Pattern 2: ETL Pipeline (Python)

**Generate ETL script:**

```
Chat → Agent mode

"Create Python ETL pipeline for customer data:
1. Extract: Read from PostgreSQL source database
2. Transform: 
   - Clean email addresses
   - Standardize phone numbers
   - Calculate customer segments (VIP, Regular, Inactive)
   - Deduplicate records
3. Load: Insert into data warehouse (Snowflake)
Include error handling, logging, and data validation"
```

**Result:**

```python
# etl_customer_pipeline.py
import pandas as pd
import psycopg2
from psycopg2.extras import RealDictCursor
import snowflake.connector
from typing import Dict, List, Optional
import logging
from datetime import datetime, timedelta
import re
from dataclasses import dataclass

# Configure logging
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    handlers=[
        logging.FileHandler(f'etl_customer_{datetime.now().strftime("%Y%m%d")}.log'),
        logging.StreamHandler()
    ]
)
logger = logging.getLogger(__name__)

@dataclass
class DatabaseConfig:
    """Database connection configuration"""
    host: str
    port: int
    database: str
    user: str
    password: str

class CustomerETL:
    """ETL pipeline for customer data"""
    
    def __init__(self, source_config: DatabaseConfig, target_config: Dict):
        self.source_config = source_config
        self.target_config = target_config
        self.errors = []
        
    def extract(self) -> pd.DataFrame:
        """Extract customer data from PostgreSQL"""
        logger.info("Starting data extraction from PostgreSQL...")
        
        try:
            conn = psycopg2.connect(
                host=self.source_config.host,
                port=self.source_config.port,
                database=self.source_config.database,
                user=self.source_config.user,
                password=self.source_config.password
            )
            
            query = """
                SELECT 
                    u.id,
                    u.email,
                    u.first_name,
                    u.last_name,
                    u.phone,
                    u.created_at,
                    u.updated_at,
                    COUNT(DISTINCT o.id) AS total_orders,
                    COALESCE(SUM(o.total), 0) AS lifetime_value,
                    MAX(o.created_at) AS last_order_date
                FROM users u
                LEFT JOIN orders o ON u.id = o.user_id 
                    AND o.status IN ('DELIVERED', 'SHIPPED')
                WHERE u.created_at >= CURRENT_DATE - INTERVAL '2 years'
                GROUP BY u.id, u.email, u.first_name, u.last_name, 
                         u.phone, u.created_at, u.updated_at
            """
            
            df = pd.read_sql_query(query, conn)
            conn.close()
            
            logger.info(f"Extracted {len(df)} records from source database")
            return df
            
        except Exception as e:
            logger.error(f"Extraction failed: {str(e)}")
            raise
    
    def transform(self, df: pd.DataFrame) -> pd.DataFrame:
        """Transform customer data"""
        logger.info("Starting data transformation...")
        
        initial_count = len(df)
        
        # Clean email addresses
        df['email'] = df['email'].apply(self._clean_email)
        
        # Standardize phone numbers
        df['phone'] = df['phone'].apply(self._standardize_phone)
        
        # Calculate customer segments
        df['segment'] = df.apply(self._calculate_segment, axis=1)
        
        # Calculate customer age in days
        df['customer_age_days'] = (
            datetime.now() - pd.to_datetime(df['created_at'])
        ).dt.days
        
        # Calculate days since last order
        df['days_since_last_order'] = df['last_order_date'].apply(
            lambda x: (datetime.now() - x).days if pd.notna(x) else None
        )
        
        # Add derived fields
        df['is_active'] = df['days_since_last_order'].apply(
            lambda x: x < 90 if pd.notna(x) else False
        )
        
        # Deduplicate by email (keep most recent)
        df = df.sort_values('updated_at', ascending=False)
        df_deduped = df.drop_duplicates(subset=['email'], keep='first')
        
        duplicates_removed = initial_count - len(df_deduped)
        if duplicates_removed > 0:
            logger.info(f"Removed {duplicates_removed} duplicate records")
        
        # Validate data
        df_validated = self._validate_data(df_deduped)
        
        logger.info(f"Transformation complete: {len(df_validated)} valid records")
        return df_validated
    
    def load(self, df: pd.DataFrame):
        """Load transformed data into Snowflake"""
        logger.info("Starting data load to Snowflake...")
        
        try:
            conn = snowflake.connector.connect(
                account=self.target_config['account'],
                user=self.target_config['user'],
                password=self.target_config['password'],
                warehouse=self.target_config['warehouse'],
                database=self.target_config['database'],
                schema=self.target_config['schema']
            )
            
            cursor = conn.cursor()
            
            # Create staging table
            cursor.execute("""
                CREATE OR REPLACE TEMPORARY TABLE customer_staging (
                    customer_id VARCHAR(255),
                    email VARCHAR(255),
                    first_name VARCHAR(255),
                    last_name VARCHAR(255),
                    phone VARCHAR(50),
                    segment VARCHAR(50),
                    total_orders INTEGER,
                    lifetime_value DECIMAL(10, 2),
                    customer_age_days INTEGER,
                    days_since_last_order INTEGER,
                    is_active BOOLEAN,
                    last_order_date TIMESTAMP,
                    created_at TIMESTAMP,
                    updated_at TIMESTAMP,
                    etl_loaded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP()
                )
            """)
            
            # Insert data into staging
            for _, row in df.iterrows():
                cursor.execute("""
                    INSERT INTO customer_staging (
                        customer_id, email, first_name, last_name, phone,
                        segment, total_orders, lifetime_value, customer_age_days,
                        days_since_last_order, is_active, last_order_date,
                        created_at, updated_at
                    ) VALUES (
                        %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s
                    )
                """, (
                    row['id'], row['email'], row['first_name'], row['last_name'],
                    row['phone'], row['segment'], row['total_orders'],
                    row['lifetime_value'], row['customer_age_days'],
                    row['days_since_last_order'], row['is_active'],
                    row['last_order_date'], row['created_at'], row['updated_at']
                ))
            
            # Merge into target table
            cursor.execute("""
                MERGE INTO customer_dimension AS target
                USING customer_staging AS source
                ON target.customer_id = source.customer_id
                WHEN MATCHED THEN
                    UPDATE SET
                        email = source.email,
                        first_name = source.first_name,
                        last_name = source.last_name,
                        phone = source.phone,
                        segment = source.segment,
                        total_orders = source.total_orders,
                        lifetime_value = source.lifetime_value,
                        customer_age_days = source.customer_age_days,
                        days_since_last_order = source.days_since_last_order,
                        is_active = source.is_active,
                        last_order_date = source.last_order_date,
                        updated_at = source.updated_at,
                        etl_loaded_at = CURRENT_TIMESTAMP()
                WHEN NOT MATCHED THEN
                    INSERT (
                        customer_id, email, first_name, last_name, phone,
                        segment, total_orders, lifetime_value, customer_age_days,
                        days_since_last_order, is_active, last_order_date,
                        created_at, updated_at, etl_loaded_at
                    ) VALUES (
                        source.customer_id, source.email, source.first_name,
                        source.last_name, source.phone, source.segment,
                        source.total_orders, source.lifetime_value,
                        source.customer_age_days, source.days_since_last_order,
                        source.is_active, source.last_order_date,
                        source.created_at, source.updated_at,
                        CURRENT_TIMESTAMP()
                    )
            """)
            
            conn.commit()
            cursor.close()
            conn.close()
            
            logger.info(f"Successfully loaded {len(df)} records to Snowflake")
            
        except Exception as e:
            logger.error(f"Load failed: {str(e)}")
            raise
    
    def _clean_email(self, email: str) -> Optional[str]:
        """Clean and validate email address"""
        if pd.isna(email):
            return None
        
        email = email.lower().strip()
        
        # Basic email validation
        email_pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
        if re.match(email_pattern, email):
            return email
        else:
            self.errors.append(f"Invalid email: {email}")
            return None
    
    def _standardize_phone(self, phone: str) -> Optional[str]:
        """Standardize phone number format"""
        if pd.isna(phone):
            return None
        
        # Remove all non-digit characters
        digits = re.sub(r'\D', '', phone)
        
        # Format as +1-XXX-XXX-XXXX for US numbers
        if len(digits) == 10:
            return f"+1-{digits[0:3]}-{digits[3:6]}-{digits[6:10]}"
        elif len(digits) == 11 and digits[0] == '1':
            return f"+1-{digits[1:4]}-{digits[4:7]}-{digits[7:11]}"
        else:
            return phone  # Return original if can't standardize
    
    def _calculate_segment(self, row: pd.Series) -> str:
        """Calculate customer segment"""
        lifetime_value = row['lifetime_value']
        total_orders = row['total_orders']
        days_since_last_order = row.get('days_since_last_order')
        
        # VIP: High lifetime value and frequent orders
        if lifetime_value >= 1000 and total_orders >= 10:
            return 'VIP'
        
        # Active: Recent orders
        elif pd.notna(days_since_last_order) and days_since_last_order < 30:
            return 'ACTIVE'
        
        # Regular: Some purchase history
        elif total_orders >= 3:
            return 'REGULAR'
        
        # Inactive: No recent orders
        elif pd.notna(days_since_last_order) and days_since_last_order > 180:
            return 'INACTIVE'
        
        # New: Few or no orders
        else:
            return 'NEW'
    
    def _validate_data(self, df: pd.DataFrame) -> pd.DataFrame:
        """Validate data quality"""
        # Remove records with invalid emails
        df_valid = df[df['email'].notna()].copy()
        
        invalid_count = len(df) - len(df_valid)
        if invalid_count > 0:
            logger.warning(f"Removed {invalid_count} records with invalid emails")
        
        # Check for required fields
        required_fields = ['id', 'email', 'first_name', 'last_name']
        for field in required_fields:
            null_count = df_valid[field].isna().sum()
            if null_count > 0:
                logger.warning(f"Found {null_count} null values in {field}")
        
        return df_valid
    
    def run(self):
        """Execute complete ETL pipeline"""
        try:
            start_time = datetime.now()
            logger.info("="*50)
            logger.info("Starting Customer ETL Pipeline")
            logger.info("="*50)
            
            # Extract
            df = self.extract()
            
            # Transform
            df_transformed = self.transform(df)
            
            # Load
            self.load(df_transformed)
            
            duration = (datetime.now() - start_time).total_seconds()
            logger.info("="*50)
            logger.info(f"ETL Pipeline completed successfully in {duration:.2f} seconds")
            logger.info(f"Total records processed: {len(df_transformed)}")
            logger.info(f"Total errors: {len(self.errors)}")
            logger.info("="*50)
            
            return {
                'status': 'success',
                'records_processed': len(df_transformed),
                'duration_seconds': duration,
                'errors': self.errors
            }
            
        except Exception as e:
            logger.error(f"ETL Pipeline failed: {str(e)}")
            return {
                'status': 'failed',
                'error': str(e),
                'errors': self.errors
            }

# Usage
if __name__ == "__main__":
    source_config = DatabaseConfig(
        host='localhost',
        port=5432,
        database='ecommerce',
        user='etl_user',
        password='password'
    )
    
    target_config = {
        'account': 'your_account',
        'user': 'etl_user',
        'password': 'password',
        'warehouse': 'ETL_WH',
        'database': 'ANALYTICS',
        'schema': 'DIM'
    }
    
    etl = CustomerETL(source_config, target_config)
    result = etl.run()
    
    print(f"ETL Result: {result}")
```

---

### Pattern 3: Database Migration Scripts

**Generate migration with rollback:**

```
Chat:

"Create PostgreSQL migration to:
1. Add user_preferences table
2. Add foreign key to users
3. Add indexes for performance
4. Include rollback script
Use Knex.js migration format"
```

**Result:**

```javascript
// migrations/20250101120000_add_user_preferences.js

/**
 * @param { import("knex").Knex } knex
 * @returns { Promise<void> }
 */
exports.up = async function(knex) {
  // Create user_preferences table
  await knex.schema.createTable('user_preferences', (table) => {
    table.uuid('id').primary().defaultTo(knex.raw('gen_random_uuid()'));
    table.uuid('user_id').notNullable();
    table.enum('theme', ['light', 'dark', 'auto']).defaultTo('light');
    table.string('language', 10).defaultTo('en');
    table.boolean('email_notifications').defaultTo(true);
    table.boolean('push_notifications').defaultTo(true);
    table.boolean('sms_notifications').defaultTo(false);
    table.jsonb('custom_settings').defaultTo('{}');
    table.timestamps(true, true);
    
    // Foreign key constraint
    table.foreign('user_id')
      .references('id')
      .inTable('users')
      .onDelete('CASCADE');
    
    // Unique constraint (one preference per user)
    table.unique(['user_id']);
  });
  
  // Add indexes for performance
  await knex.schema.raw(`
    CREATE INDEX idx_user_preferences_user_id 
    ON user_preferences(user_id);
    
    CREATE INDEX idx_user_preferences_theme 
    ON user_preferences(theme);
    
    CREATE INDEX idx_user_preferences_language 
    ON user_preferences(language);
  `);
  
  // Create default preferences for existing users
  await knex.raw(`
    INSERT INTO user_preferences (user_id, theme, language)
    SELECT 
      id,
      'light',
      'en'
    FROM users
    WHERE NOT EXISTS (
      SELECT 1 
      FROM user_preferences up 
      WHERE up.user_id = users.id
    );
  `);
  
  console.log('✅ Migration: user_preferences table created successfully');
};

/**
 * @param { import("knex").Knex } knex
 * @returns { Promise<void> }
 */
exports.down = async function(knex) {
  // Drop indexes first
  await knex.schema.raw(`
    DROP INDEX IF EXISTS idx_user_preferences_user_id;
    DROP INDEX IF EXISTS idx_user_preferences_theme;
    DROP INDEX IF EXISTS idx_user_preferences_language;
  `);
  
  // Drop table (CASCADE will handle foreign key)
  await knex.schema.dropTableIfExists('user_preferences');
  
  console.log('✅ Rollback: user_preferences table removed successfully');
};
```

**Prisma migration:**

```
Chat:

"Generate Prisma migration for adding full-text search to products:
- Add tsvector column
- Create GIN index
- Add trigger to auto-update search vector
Include both Prisma schema and raw SQL"
```

**Result:**

```prisma
// schema.prisma update
model Product {
  id          String   @id @default(uuid())
  name        String
  description String?  @db.Text
  price       Decimal  @db.Decimal(10, 2)
  stock       Int      @default(0)
  searchVector String?  @db.Text  // For full-text search
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  @@index([name])
  @@map("products")
}
```

```sql
-- migrations/add_fulltext_search/migration.sql
-- Add tsvector column for full-text search
ALTER TABLE products 
ADD COLUMN search_vector tsvector;

-- Create function to update search vector
CREATE OR REPLACE FUNCTION products_search_vector_update() 
RETURNS TRIGGER AS $$
BEGIN
  NEW.search_vector := 
    setweight(to_tsvector('english', COALESCE(NEW.name, '')), 'A') ||
    setweight(to_tsvector('english', COALESCE(NEW.description, '')), 'B');
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Create trigger
CREATE TRIGGER products_search_vector_trigger
BEFORE INSERT OR UPDATE ON products
FOR EACH ROW
EXECUTE FUNCTION products_search_vector_update();

-- Create GIN index for fast full-text search
CREATE INDEX products_search_vector_idx 
ON products USING GIN(search_vector);

-- Update existing rows
UPDATE products 
SET search_vector = 
  setweight(to_tsvector('english', COALESCE(name, '')), 'A') ||
  setweight(to_tsvector('english', COALESCE(description, '')), 'B');

-- Example search query
COMMENT ON COLUMN products.search_vector IS 
'Full-text search vector. Usage: SELECT * FROM products WHERE search_vector @@ to_tsquery(''search & term'');';
```

---

### Pattern 4: Data Validation & Quality Checks

**Generate data quality script:**

```
Chat:

"Create Python script for data quality validation:
- Check for null values in required fields
- Validate email formats
- Check for duplicate records
- Validate numeric ranges (price > 0, stock >= 0)
- Generate data quality report
Use Great Expectations framework"
```

**Result:**

```python
# data_quality_checks.py
import great_expectations as gx
from great_expectations.checkpoint import Checkpoint
from great_expectations.data_context import DataContext
import pandas as pd
from typing import Dict, List
import logging
from datetime import datetime

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class DataQualityValidator:
    """Data quality validation using Great Expectations"""
    
    def __init__(self, context_root_dir: str = "./gx"):
        self.context = gx.get_context(context_root_dir=context_root_dir)
        
    def validate_customers(self, df: pd.DataFrame) -> Dict:
        """Validate customer data quality"""
        logger.info("Starting customer data validation...")
        
        # Create datasource
        datasource = self.context.sources.add_or_update_pandas("customer_datasource")
        data_asset = datasource.add_dataframe_asset(name="customers")
        batch_request = data_asset.build_batch_request(dataframe=df)
        
        # Create expectation suite
        suite = self.context.add_or_update_expectation_suite("customer_quality_suite")
        
        # Define expectations
        validator = self.context.get_validator(
            batch_request=batch_request,
            expectation_suite_name="customer_quality_suite"
        )
        
        # Required fields should not be null
        validator.expect_column_values_to_not_be_null("id")
        validator.expect_column_values_to_not_be_null("email")
        validator.expect_column_values_to_not_be_null("first_name")
        validator.expect_column_values_to_not_be_null("last_name")
        
        # Email format validation
        validator.expect_column_values_to_match_regex(
            "email",
            regex=r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
        )
        
        # No duplicate emails
        validator.expect_column_values_to_be_unique("email")
        
        # Numeric validations
        validator.expect_column_values_to_be_between(
            "total_orders",
            min_value=0,
            max_value=None
        )
        
        validator.expect_column_values_to_be_between(
            "lifetime_value",
            min_value=0,
            max_value=None
        )
        
        # Date validations
        validator.expect_column_values_to_be_of_type("created_at", "datetime64")
        validator.expect_column_values_to_be_dateutil_parseable("created_at")
        
        # Run validation
        results = validator.validate()
        
        # Generate report
        report = self._generate_report(results)
        
        return report
    
    def validate_products(self, df: pd.DataFrame) -> Dict:
        """Validate product data quality"""
        logger.info("Starting product data validation...")
        
        datasource = self.context.sources.add_or_update_pandas("product_datasource")
        data_asset = datasource.add_dataframe_asset(name="products")
        batch_request = data_asset.build_batch_request(dataframe=df)
        
        validator = self.context.get_validator(
            batch_request=batch_request,
            expectation_suite_name="product_quality_suite"
        )
        
        # Required fields
        validator.expect_column_values_to_not_be_null("id")
        validator.expect_column_values_to_not_be_null("name")
        validator.expect_column_values_to_not_be_null("price")
        
        # No duplicate product names
        validator.expect_column_values_to_be_unique("name")
        
        # Price validations
        validator.expect_column_values_to_be_between(
            "price",
            min_value=0.01,
            max_value=100000
        )
        
        # Stock validations
        validator.expect_column_values_to_be_between(
            "stock",
            min_value=0,
            max_value=None
        )
        
        # Name length
        validator.expect_column_value_lengths_to_be_between(
            "name",
            min_value=3,
            max_value=200
        )
        
        results = validator.validate()
        report = self._generate_report(results)
        
        return report
    
    def _generate_report(self, results) -> Dict:
        """Generate data quality report"""
        report = {
            'timestamp': datetime.now().isoformat(),
            'success': results.success,
            'statistics': {
                'evaluated_expectations': results.statistics['evaluated_expectations'],
                'successful_expectations': results.statistics['successful_expectations'],
                'unsuccessful_expectations': results.statistics['unsuccessful_expectations'],
                'success_percent': results.statistics['success_percent']
            },
            'failed_expectations': []
        }
        
        # Collect failed expectations
        for result in results.results:
            if not result.success:
                report['failed_expectations'].append({
                    'expectation_type': result.expectation_config.expectation_type,
                    'column': result.expectation_config.kwargs.get('column'),
                    'unexpected_count': result.result.get('unexpected_count', 0),
                    'unexpected_percent': result.result.get('unexpected_percent', 0)
                })
        
        # Log summary
        if results.success:
            logger.info(f"✅ Data quality validation PASSED ({report['statistics']['success_percent']}%)")
        else:
            logger.warning(f"❌ Data quality validation FAILED ({report['statistics']['success_percent']}%)")
            logger.warning(f"Failed expectations: {len(report['failed_expectations'])}")
        
        return report

# Usage
if __name__ == "__main__":
    # Load data
    customers_df = pd.read_csv('customers.csv')
    products_df = pd.read_csv('products.csv')
    
    # Validate
    validator = DataQualityValidator()
    
    customer_report = validator.validate_customers(customers_df)
    product_report = validator.validate_products(products_df)
    
    # Print reports
    print("\n=== Customer Data Quality Report ===")
    print(f"Success: {customer_report['success']}")
    print(f"Success Rate: {customer_report['statistics']['success_percent']}%")
    
    if customer_report['failed_expectations']:
        print("\nFailed Checks:")
        for failure in customer_report['failed_expectations']:
            print(f"  - {failure['expectation_type']} on {failure['column']}")
            print(f"    Unexpected: {failure['unexpected_count']} ({failure['unexpected_percent']}%)")
    
    print("\n=== Product Data Quality Report ===")
    print(f"Success: {product_report['success']}")
    print(f"Success Rate: {product_report['statistics']['success_percent']}%")
```

---

## 📊 Query Optimization

### Pattern 5: Analyze & Optimize Slow Queries

**Chat prompt:**

```
"Analyze this slow PostgreSQL query and suggest optimizations:

SELECT u.email, COUNT(o.id) as order_count, SUM(o.total) as total_spent
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.created_at >= '2024-01-01'
GROUP BY u.email
HAVING COUNT(o.id) > 5
ORDER BY total_spent DESC;

Include:
- Index recommendations
- Query rewrite suggestions
- EXPLAIN ANALYZE interpretation"
```

**Copilot suggests:**

```sql
-- Original query issues:
-- 1. WHERE clause on LEFT JOIN turns it into INNER JOIN
-- 2. No indexes on orders.created_at or orders.user_id
-- 3. GROUP BY on non-indexed column (email)
-- 4. HAVING filter could be optimized

-- Step 1: Add indexes
CREATE INDEX CONCURRENTLY idx_orders_user_id ON orders(user_id);
CREATE INDEX CONCURRENTLY idx_orders_created_at ON orders(created_at);
CREATE INDEX CONCURRENTLY idx_orders_user_created ON orders(user_id, created_at);
CREATE INDEX CONCURRENTLY idx_users_email ON users(email);

-- Step 2: Optimized query
-- Use INNER JOIN since we filter on orders
-- Filter early with subquery
-- Use composite index
WITH user_orders AS (
    SELECT 
        user_id,
        COUNT(*) as order_count,
        SUM(total) as total_spent
    FROM orders
    WHERE created_at >= '2024-01-01'
    GROUP BY user_id
    HAVING COUNT(*) > 5
)
SELECT 
    u.email,
    uo.order_count,
    uo.total_spent
FROM user_orders uo
INNER JOIN users u ON uo.user_id = u.id
ORDER BY uo.total_spent DESC;

-- Step 3: Materialized view for frequent queries
CREATE MATERIALIZED VIEW user_order_stats AS
SELECT 
    u.id as user_id,
    u.email,
    COUNT(o.id) as order_count,
    SUM(o.total) as total_spent,
    AVG(o.total) as avg_order_value,
    MAX(o.created_at) as last_order_date
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.created_at >= CURRENT_DATE - INTERVAL '1 year'
GROUP BY u.id, u.email;

CREATE UNIQUE INDEX idx_user_order_stats_user_id 
ON user_order_stats(user_id);

CREATE INDEX idx_user_order_stats_total_spent 
ON user_order_stats(total_spent DESC);

-- Refresh strategy
CREATE OR REPLACE FUNCTION refresh_user_order_stats()
RETURNS void AS $$
BEGIN
    REFRESH MATERIALIZED VIEW CONCURRENTLY user_order_stats;
END;
$$ LANGUAGE plpgsql;

-- Schedule refresh (using pg_cron)
SELECT cron.schedule('refresh-user-stats', '0 2 * * *', 
  'SELECT refresh_user_order_stats()');

-- Query the materialized view
SELECT 
    email,
    order_count,
    total_spent
FROM user_order_stats
WHERE order_count > 5
ORDER BY total_spent DESC;

-- Performance comparison
EXPLAIN (ANALYZE, BUFFERS) 
-- Original query
vs
EXPLAIN (ANALYZE, BUFFERS)
-- Optimized query

/*
Expected improvements:
- Original: ~2000ms, Seq Scan on orders
- Optimized: ~50ms, Index Scan on idx_orders_user_created
- Materialized view: ~5ms, Index Scan on idx_user_order_stats_total_spent
*/
```

---

## 🎯 Custom Instructions for Data Engineering

Create `.github/copilot-instructions.md`:

```markdown
# Data Engineering Standards

## SQL Conventions
- Always use explicit column names (no SELECT *)
- Use CTEs (WITH) for complex queries
- Add EXPLAIN ANALYZE for optimization
- Include comments for complex logic
- Use parameterized queries (prevent SQL injection)

## Query Performance
- Create indexes on:
  - Foreign keys
  - WHERE clause columns
  - JOIN columns
  - ORDER BY columns
- Avoid N+1 queries
- Use LIMIT for large result sets
- Consider materialized views for expensive aggregations

## ETL Patterns
- Extract: Read in batches to manage memory
- Transform: Validate data at each step
- Load: Use UPSERT/MERGE for idempotency
- Always include error handling and logging
- Support incremental updates (not just full loads)

## Data Quality
- Validate all data before loading
- Check for nulls in required fields
- Verify data types and ranges
- Deduplicate records
- Log data quality metrics

## Migration Safety
- Always include rollback scripts
- Test migrations on staging first
- Use transactions for atomic changes
- Add indexes CONCURRENTLY in PostgreSQL
- Backup data before destructive operations
```

---

## 💡 Pro Tips

### 1. **Generate SQL from Natural Language**

```
Chat:

"Convert to SQL: Get top 5 products by revenue in the last quarter, 
broken down by category. Include product name, category, total quantity sold, 
and revenue. Database is PostgreSQL."
```

### 2. **dbt Model Generation**

```
Chat:

"Create dbt model for customer lifetime value:
- Base model: stg_customers, stg_orders
- Aggregate orders per customer
- Calculate LTV, AOV, frequency
- Include tests for uniqueness and not_null"
```

### 3. **Airflow DAG for ETL**

```
Chat:

"Create Airflow DAG for daily customer ETL:
- Task 1: Extract from PostgreSQL
- Task 2: Transform data (Python)
- Task 3: Load to Snowflake
- Task 4: Run data quality checks
- Task 5: Send notification
Include retries and SLA monitoring"
```

---

## 🚨 Common Pitfalls

### ❌ Don't:
- ❌ Use SELECT * in production queries
- ❌ Skip indexes on foreign keys
- ❌ Load entire tables into memory
- ❌ Hardcode connection strings
- ❌ Forget to handle NULL values
- ❌ Run migrations without rollback plans

### ✅ Do:
- ✅ Use batching for large datasets
- ✅ Add proper indexes
- ✅ Validate data quality
- ✅ Log ETL metrics
- ✅ Use transactions for consistency
- ✅ Test migrations on staging first

---

## 📚 Related Resources

**Official Documentation:**
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [MySQL Documentation](https://dev.mysql.com/doc/)
- [Snowflake Docs](https://docs.snowflake.com/)
- [Apache Airflow](https://airflow.apache.org/docs/)
- [dbt Documentation](https://docs.getdbt.com/)
- [Great Expectations](https://docs.greatexpectations.io/)

**Copilot Features Used:**
- Inline completions for SQL queries
- Chat for complex ETL pipelines
- Agent mode for multi-file data scripts
- Custom instructions for SQL conventions

---

## 🎯 Next Steps

- **Continue to:** [DevOps & Infrastructure →](04-devops-infrastructure.md)
- **Related:** [Backend Development](02-backend-development.md)
- **Practice:** Build ETL pipeline with data quality checks
- **Experiment:** Create custom SQL optimization agent

---

**Updated:** December 2025  
**Part 5, Section 3 of 4**

