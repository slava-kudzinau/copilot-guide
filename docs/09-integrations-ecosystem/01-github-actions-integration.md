---
title: "Section 1: GitHub Actions Integration"
parent: "Part 9: Integrations & Ecosystem"
nav_order: 1
---

# Section 1: GitHub Actions Integration

**Automate your development workflow with GitHub Copilot and Actions**

---

## 📋 Overview

GitHub Actions provides powerful CI/CD automation capabilities, and GitHub Copilot can significantly accelerate workflow creation, maintenance, and optimization. This section explores how to leverage Copilot to generate Actions workflows, automate Copilot-related tasks, and build Actions that integrate with Copilot's capabilities.

**What you'll learn:**
- Using Copilot to generate GitHub Actions workflows
- Automating Copilot license management with Actions
- Building custom Actions that leverage Copilot
- Best practices for secure and efficient workflows
- Monitoring and optimizing Actions performance

**Time to complete:** 1-2 hours

---

## 🎯 Use Cases

### 1. Workflow Generation with Copilot

**Scenario:** You need to create a CI/CD pipeline for a Node.js application.

**Copilot Prompt:**

```
@workspace Generate a GitHub Actions workflow that:
- Triggers on push to main and pull requests
- Runs on Ubuntu latest
- Sets up Node.js 20 with npm caching
- Installs dependencies with npm ci
- Runs linting with npm run lint
- Runs tests with npm test
- Uploads test coverage to Codecov
- Deploys to production if on main branch and tests pass
```

**Generated Workflow:**

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linting
        run: npm run lint
      
      - name: Run tests
        run: npm test
      
      - name: Upload coverage
        uses: codecov/codecov-action@v4
        with:
          token: ${{ secrets.CODECOV_TOKEN }}
          fail_ci_if_error: true
  
  deploy:
    needs: test
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Deploy to production
        run: npm run deploy
        env:
          DEPLOY_TOKEN: ${{ secrets.DEPLOY_TOKEN }}
```

**Key Features:**
- ✅ Proper trigger configuration
- ✅ Node.js caching for faster builds
- ✅ Separate test and deploy jobs
- ✅ Conditional deployment
- ✅ Timeout protection
- ✅ Secure secret handling

---

### 2. Copilot License Management Automation

**Scenario:** Track inactive Copilot users and send reminders to optimize license usage.

**Workflow: `.github/workflows/copilot-license-reminders.yml`**

```yaml
name: Copilot License Reminders

on:
  schedule:
    - cron: '0 8 * * 1' # Every Monday at 8 AM UTC
  workflow_dispatch: # Allow manual trigger

jobs:
  remind-inactive-users:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      issues: write
    
    steps:
      - name: Check last Copilot activity
        id: check-activity
        run: |
          # Fetch Copilot seat assignments
          RESPONSE=$(gh api \
            -H "Accept: application/vnd.github+json" \
            -H "X-GitHub-Api-Version: 2022-11-28" \
            -H "Authorization: Bearer ${{ secrets.COPILOT_LICENSE_READ }}" \
            /orgs/${{ github.repository_owner }}/copilot/billing/seats)
          
          echo "Raw Response:"
          echo "$RESPONSE"
          
          # Process each user
          echo "$RESPONSE" | jq -c '.seats[]' | while read -r seat; do
            LOGIN=$(echo "$seat" | jq -r '.assignee.login')
            LAST_ACTIVITY=$(echo "$seat" | jq -r '.last_activity_at')
            CREATED_AT=$(echo "$seat" | jq -r '.created_at')
            
            # Check for existing reminder issues
            EXISTING_ISSUES=$(gh issue list \
              --repo ${{ github.repository }} \
              --assignee $LOGIN \
              --label 'copilot-reminder' \
              --json id)
            
            # Calculate activity date
            if [ "$LAST_ACTIVITY" = "null" ]; then
              LAST_ACTIVITY_DATE=$(date -d "$CREATED_AT" +%s)
            else
              LAST_ACTIVITY_DATE=$(date -d "$LAST_ACTIVITY" +%s)
            fi
            
            THIRTY_DAYS_AGO=$(date -d "30 days ago" +%s)
            
            # Create issue for inactive users
            if [ "$LAST_ACTIVITY_DATE" -lt "$THIRTY_DAYS_AGO" ] && [ "$EXISTING_ISSUES" = "[]" ]; then
              echo "User $LOGIN inactive for 30+ days"
              
              gh issue create \
                --title "Reminder: GitHub Copilot License" \
                --body "${{ vars.COPILOT_REMINDER_MESSAGE }}" \
                --repo ${{ github.repository }} \
                --assignee $LOGIN \
                --label 'copilot-reminder'
            else
              echo "User $LOGIN is active or has existing reminder"
            fi
          done
        env:
          GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

**Setup Requirements:**

1. **Create GitHub Secret:** `COPILOT_LICENSE_READ`
   - Go to Settings → Secrets and variables → Actions
   - Create new secret with a personal access token that has `manage_billing:copilot` scope

2. **Create Repository Variable:** `COPILOT_REMINDER_MESSAGE`
   - Go to Settings → Secrets and variables → Actions → Variables
   - Example message:
   ```markdown
   Hi! 👋
   
   We noticed you haven't used your GitHub Copilot license in the last 30 days.
   
   **Action Required:**
   - If you're still using Copilot, great! This reminder will auto-close.
   - If you're not using it, please let us know so we can reassign the license.
   
   Questions? Reach out to the dev tools team.
   ```

3. **Grant Permissions:**
   - Ensure workflow has `contents: read` and `issues: write` permissions

---

### 3. Copilot Usage Analytics Collection

**Scenario:** Track Copilot usage metrics for your organization.

**Workflow: `.github/workflows/copilot-analytics.yml`**

```yaml
name: Copilot Usage Analytics

on:
  schedule:
    - cron: '0 0 * * *' # Daily at midnight UTC
  workflow_dispatch:

jobs:
  collect-metrics:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    
    steps:
      - name: Checkout repository
        uses: actions/checkout@v4
      
      - name: Fetch Copilot metrics
        id: fetch-metrics
        run: |
          # Fetch usage data
          gh api \
            -H "Accept: application/vnd.github+json" \
            -H "X-GitHub-Api-Version: 2022-11-28" \
            /orgs/${{ github.repository_owner }}/copilot/usage \
            > metrics-$(date +%Y-%m-%d).json
          
          # Fetch seat assignments
          gh api \
            -H "Accept: application/vnd.github+json" \
            -H "X-GitHub-Api-Version: 2022-11-28" \
            /orgs/${{ github.repository_owner }}/copilot/billing/seats \
            > seats-$(date +%Y-%m-%d).json
        env:
          GH_TOKEN: ${{ secrets.COPILOT_METRICS_TOKEN }}
      
      - name: Process and store metrics
        run: |
          # Create analytics directory if it doesn't exist
          mkdir -p .github/copilot-analytics
          
          # Move files to analytics directory
          mv metrics-*.json .github/copilot-analytics/
          mv seats-*.json .github/copilot-analytics/
          
          # Generate summary report
          cat > .github/copilot-analytics/latest-summary.md << 'EOF'
          # Copilot Usage Summary
          **Generated:** $(date)
          
          ## Metrics Collected
          - Daily usage data
          - Seat assignments
          - Activity patterns
          
          See JSON files for detailed data.
          EOF
      
      - name: Commit metrics
        run: |
          git config user.name "GitHub Actions"
          git config user.email "actions@github.com"
          git add .github/copilot-analytics/
          git commit -m "chore: update Copilot usage metrics [skip ci]" || echo "No changes"
          git push
```

**Analysis Script:** `scripts/analyze-copilot-metrics.js`

```javascript
const fs = require('fs');
const path = require('path');

// Read latest metrics
const metricsDir = '.github/copilot-analytics';
const files = fs.readdirSync(metricsDir)
  .filter(f => f.startsWith('metrics-'))
  .sort()
  .reverse();

const latestMetrics = JSON.parse(
  fs.readFileSync(path.join(metricsDir, files[0]), 'utf8')
);

// Calculate key metrics
const totalUsers = latestMetrics.length;
const activeUsers = latestMetrics.filter(u => u.total_suggestions_count > 0).length;
const adoptionRate = (activeUsers / totalUsers * 100).toFixed(1);

const totalSuggestions = latestMetrics.reduce(
  (sum, u) => sum + u.total_suggestions_count, 0
);
const acceptedSuggestions = latestMetrics.reduce(
  (sum, u) => sum + u.total_acceptances_count, 0
);
const acceptanceRate = (acceptedSuggestions / totalSuggestions * 100).toFixed(1);

console.log('Copilot Usage Analytics');
console.log('======================');
console.log(`Total Users: ${totalUsers}`);
console.log(`Active Users: ${activeUsers}`);
console.log(`Adoption Rate: ${adoptionRate}%`);
console.log(`Total Suggestions: ${totalSuggestions}`);
console.log(`Accepted Suggestions: ${acceptedSuggestions}`);
console.log(`Acceptance Rate: ${acceptanceRate}%`);
```

---

## 🔧 Custom Instructions for Actions Workflows

**Create:** `.github/copilot-instructions.md`

```markdown
# GitHub Actions Workflow Guidelines

When generating or modifying GitHub Actions workflows:

## Security Requirements
- Pin all third-party actions to specific commit SHAs
- Use GitHub secrets for all sensitive data
- Configure minimal GITHUB_TOKEN permissions
- Never log secrets or sensitive information
- Use `secrets.inherit` sparingly

## Performance Best Practices
- Always cache dependencies (npm, pip, cargo, etc.)
- Add `timeout-minutes` to all jobs (default: 10-15 minutes)
- Use matrix strategies for parallel execution
- Minimize checkout depth with `fetch-depth: 1` when possible
- Use `actions/cache` for build artifacts

## Workflow Structure
- Use descriptive names for workflows, jobs, and steps
- Include both `push` and `pull_request` triggers
- Add `workflow_dispatch` for manual triggering
- Separate concerns into multiple jobs
- Use job dependencies with `needs`

## Error Handling
- Add `if: always()` for cleanup steps
- Use `continue-on-error: true` for non-critical steps
- Include failure notifications (Slack, email, etc.)
- Log detailed error information

## Example Template
```yaml
name: Descriptive Workflow Name

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:

jobs:
  job-name:
    runs-on: ubuntu-latest
    timeout-minutes: 10
    permissions:
      contents: read
    
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 1
      
      - name: Descriptive step name
        run: |
          # Commands here
```
```

---

## 🎨 Workflow Patterns

### Pattern 1: Multi-Environment Deployment

```yaml
name: Multi-Environment Deploy

on:
  push:
    branches: [ main, staging, develop ]

jobs:
  determine-environment:
    runs-on: ubuntu-latest
    outputs:
      environment: ${{ steps.set-env.outputs.environment }}
    
    steps:
      - id: set-env
        run: |
          if [ "${{ github.ref }}" = "refs/heads/main" ]; then
            echo "environment=production" >> $GITHUB_OUTPUT
          elif [ "${{ github.ref }}" = "refs/heads/staging" ]; then
            echo "environment=staging" >> $GITHUB_OUTPUT
          else
            echo "environment=development" >> $GITHUB_OUTPUT
          fi
  
  deploy:
    needs: determine-environment
    runs-on: ubuntu-latest
    environment: ${{ needs.determine-environment.outputs.environment }}
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to ${{ needs.determine-environment.outputs.environment }}
        run: |
          echo "Deploying to ${{ needs.determine-environment.outputs.environment }}"
          # Deployment commands
```

---

### Pattern 2: Reusable Workflow

**File:** `.github/workflows/reusable-test.yml`

```yaml
name: Reusable Test Workflow

on:
  workflow_call:
    inputs:
      node-version:
        required: true
        type: string
      working-directory:
        required: false
        type: string
        default: '.'
    secrets:
      npm-token:
        required: false

jobs:
  test:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ${{ inputs.working-directory }}
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
        env:
          NPM_TOKEN: ${{ secrets.npm-token }}
      
      - name: Run tests
        run: npm test
```

**Usage:** `.github/workflows/ci.yml`

```yaml
name: CI

on: [push, pull_request]

jobs:
  test-node-18:
    uses: ./.github/workflows/reusable-test.yml
    with:
      node-version: '18'
  
  test-node-20:
    uses: ./.github/workflows/reusable-test.yml
    with:
      node-version: '20'
```

---

### Pattern 3: Copilot Setup for Custom Agents

**File:** `.github/workflows/copilot-setup.yml`

```yaml
name: Copilot Development Environment Setup

on:
  workflow_dispatch:
  push:
    paths:
      - '.github/workflows/copilot-setup.yml'
      - '.github/copilot-instructions.md'
  pull_request:
    paths:
      - '.github/workflows/copilot-setup.yml'

jobs:
  setup-and-validate:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          lfs: true # Enable Git LFS if needed
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Validate Copilot instructions
        run: |
          if [ -f ".github/copilot-instructions.md" ]; then
            echo "✅ Copilot instructions found"
            wc -l .github/copilot-instructions.md
          else
            echo "⚠️  No Copilot instructions file"
          fi
      
      - name: Validate custom agents
        run: |
          if [ -d ".vscode/agents" ]; then
            echo "✅ Custom agents found:"
            ls -la .vscode/agents/
          else
            echo "⚠️  No custom agents directory"
          fi
      
      - name: Run project-specific setup
        run: |
          # Add any project-specific setup commands
          echo "Project setup complete"
```

---

## 🔒 Security Best Practices

### 1. Pin Actions to Commit SHAs

❌ **Bad:**
```yaml
- uses: actions/checkout@v4
```

✅ **Good:**
```yaml
- uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11 # v4.1.1
```

**Why:** Prevents supply chain attacks if action repository is compromised.

---

### 2. Minimal GITHUB_TOKEN Permissions

❌ **Bad:**
```yaml
permissions: write-all
```

✅ **Good:**
```yaml
permissions:
  contents: read
  issues: write
  pull-requests: write
```

---

### 3. Secret Management

✅ **Best Practices:**
- Store all credentials in GitHub Secrets
- Use environment-specific secrets
- Rotate secrets regularly
- Never log secret values
- Use `secrets.inherit` only when necessary

**Example:**
```yaml
steps:
  - name: Deploy
    run: ./deploy.sh
    env:
      API_KEY: ${{ secrets.DEPLOY_API_KEY }}
      # Never do: echo $API_KEY
```

---

## 📊 Monitoring and Optimization

### Workflow Performance Metrics

**Track these metrics:**
- ✅ Workflow duration
- ✅ Job success/failure rates
- ✅ Cache hit rates
- ✅ Concurrent workflow usage
- ✅ Cost per workflow run

**Optimization Techniques:**

1. **Caching:**
```yaml
- uses: actions/cache@v4
  with:
    path: |
      ~/.npm
      ~/.cache
      node_modules
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-
```

2. **Parallel Jobs:**
```yaml
jobs:
  test:
    strategy:
      matrix:
        node-version: [18, 20, 22]
        os: [ubuntu-latest, windows-latest, macos-latest]
    runs-on: ${{ matrix.os }}
```

3. **Conditional Execution:**
```yaml
- name: Deploy
  if: github.ref == 'refs/heads/main' && github.event_name == 'push'
  run: npm run deploy
```

---

## 🎯 Real-World Examples

### Example 1: Full-Stack Application CI/CD

```yaml
name: Full-Stack CI/CD

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  frontend-test:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./frontend
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
          cache-dependency-path: frontend/package-lock.json
      
      - run: npm ci
      - run: npm run lint
      - run: npm test
      - run: npm run build
      
      - uses: actions/upload-artifact@v4
        with:
          name: frontend-build
          path: frontend/dist
  
  backend-test:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: ./backend
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
          cache-dependency-path: backend/package-lock.json
      
      - run: npm ci
      - run: npm run lint
      - run: npm test
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test
  
  deploy:
    needs: [frontend-test, backend-test]
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    environment: production
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/download-artifact@v4
        with:
          name: frontend-build
          path: frontend/dist
      
      - name: Deploy to production
        run: |
          # Deployment commands
          echo "Deploying to production"
```

---

## 🚀 Advanced Patterns

### Custom Action with Copilot Integration

**File:** `.github/actions/copilot-review/action.yml`

```yaml
name: 'Copilot Code Review'
description: 'Automated code review using GitHub Copilot'
inputs:
  github-token:
    description: 'GitHub token'
    required: true
  files:
    description: 'Files to review (glob pattern)'
    required: false
    default: '**/*.{js,ts,jsx,tsx,py}'

runs:
  using: 'composite'
  steps:
    - name: Review code
      shell: bash
      run: |
        echo "Running Copilot code review on: ${{ inputs.files }}"
        # Integration with Copilot API would go here
        # This is a placeholder for demonstration
```

---

## 📚 Additional Resources

### Official Documentation
- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [GitHub Copilot for Actions](https://docs.github.com/en/copilot/using-github-copilot/using-github-copilot-in-the-command-line)
- [Workflow Syntax Reference](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)

### Community Resources
- [Awesome Actions](https://github.com/sdras/awesome-actions)
- [Actions Marketplace](https://github.com/marketplace?type=actions)

---

## ✅ Checklist: Actions Integration

Before deploying workflows:

- [ ] All actions pinned to commit SHAs
- [ ] Minimal GITHUB_TOKEN permissions configured
- [ ] Secrets stored in GitHub Secrets (never in code)
- [ ] Timeout configured for all jobs
- [ ] Caching implemented for dependencies
- [ ] Error handling and notifications configured
- [ ] Workflow tested with `workflow_dispatch`
- [ ] Documentation updated
- [ ] Team trained on new workflows

---

## 🎓 Practice Exercises

### Exercise 1: Generate CI Workflow
**Goal:** Use Copilot to generate a complete CI workflow  
**Time:** 15 minutes

1. Open Copilot Chat
2. Describe your project's CI needs
3. Review and customize generated workflow
4. Test with `workflow_dispatch`
5. Deploy to your repository

---

### Exercise 2: License Management Automation
**Goal:** Implement automated license reminder workflow  
**Time:** 30 minutes

1. Copy license reminder workflow
2. Configure secrets and variables
3. Test manually with `workflow_dispatch`
4. Enable scheduled runs
5. Monitor first execution

---

### Exercise 3: Custom Metrics Dashboard
**Goal:** Build analytics collection workflow  
**Time:** 1 hour

1. Set up metrics collection workflow
2. Create analysis script
3. Generate visualization
4. Share with team
5. Iterate based on feedback

---

**Next:** [Section 2: GitHub Issues and Projects Integration](02-github-issues-projects.md)

---

**Last Updated:** December 2025  
**Version:** 1.0

