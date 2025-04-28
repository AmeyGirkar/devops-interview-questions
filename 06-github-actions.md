# GitHub Actions Interview Questions: Beginner to Advanced

## Beginner Level Questions

### 1. What is GitHub Actions?
**Answer:** GitHub Actions is a tool built into GitHub that helps you automate tasks in your software development process. It lets you set up workflows that automatically run when specific events happen in your repository, like when someone pushes code or creates a pull request.

### 2. What is a workflow file and where do you store it?
**Answer:** A workflow file is a YAML file that describes the steps your automation should take. You store it in a special folder called `.github/workflows` in your repository. GitHub automatically looks for files in this folder.

### 3. What are the basic components of a GitHub Actions workflow?
**Answer:** 
- **Events:** Triggers that start your workflow (like push, pull request)
- **Jobs:** Groups of steps that run on the same runner
- **Steps:** Individual tasks that can run commands or actions
- **Actions:** Pre-packaged, reusable units of code
- **Runners:** Servers that run your workflows

### 4. How do you specify when a workflow should run?
**Answer:** You use the `on:` section in your workflow file to specify events that trigger the workflow. For example:
```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
```
This runs the workflow when someone pushes to the main branch or creates a pull request to the main branch.

### 5. What's the difference between `uses` and `run` in workflow steps?
**Answer:** 
- `uses` lets you include a pre-built action in your workflow (like `uses: actions/checkout@v3`)
- `run` lets you run command-line commands directly (like `run: npm test`)

## Intermediate Level Questions

### 6. How do you use environment variables in GitHub Actions?
**Answer:** You can define environment variables at different levels:
- Workflow level: Using `env:` at the top of your workflow file
- Job level: Using `env:` under a specific job
- Step level: Using `env:` under a specific step

You can access them using `${{ env.VARIABLE_NAME }}` or in shell commands with `$VARIABLE_NAME`.

### 7. What are GitHub Secrets and how do you use them?
**Answer:** Secrets are encrypted environment variables that you store in your repository or organization. They're perfect for sensitive data like API keys. You create them in your repository settings and use them in workflows like this:
```yaml
steps:
  - name: Use a secret
    run: echo "Using secret"
    env:
      API_TOKEN: ${{ secrets.API_TOKEN }}
```

### 8. How do you make jobs run one after another?
**Answer:** You use the `needs` keyword to create dependencies between jobs. For example:
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Building..."
  
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying..."
```
Here, the `deploy` job will only run after the `build` job completes successfully.

### 9. What is a matrix strategy and when would you use it?
**Answer:** A matrix strategy lets you run a job multiple times with different configurations. It's useful when you need to test on multiple operating systems or with different versions of a language. For example:
```yaml
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node-version: [14, 16, 18]
    steps:
      - uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm test
```
This runs your tests across 3 operating systems and 3 Node.js versions, for a total of 9 combinations.

### 10. How do you share data between jobs?
**Answer:** You can use artifacts to share files between jobs:
```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Build app
        run: npm run build
      - name: Upload build folder
        uses: actions/upload-artifact@v3
        with:
          name: build-files
          path: build/
  
  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Download build folder
        uses: actions/download-artifact@v3
        with:
          name: build-files
      - name: Deploy
        run: ./deploy.sh
```

## Advanced Level Questions

### 11. What are composite actions and how do they help?
**Answer:** Composite actions let you combine multiple steps into a single, reusable action. You create them by making an `action.yml` file that defines the steps. They're useful when you have a sequence of steps you want to reuse across different workflows.

### 12. How do you create reusable workflows?
**Answer:** You can create reusable workflows by using the `workflow_call` event. This allows one workflow to be called from another workflow. The calling workflow can pass inputs and secrets to the called workflow:
```yaml
# Reusable workflow in .github/workflows/reusable.yml
on:
  workflow_call:
    inputs:
      environment:
        required: true
        type: string

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - run: echo "Deploying to ${{ inputs.environment }}"
```

```yaml
# Caller workflow
jobs:
  call-reusable:
    uses: ./.github/workflows/reusable.yml
    with:
      environment: 'production'
```

### 13. How do you implement custom runners and why would you use them?
**Answer:** Self-hosted runners are servers you manage yourself that can run GitHub Actions workflows. You set them up by installing the GitHub Actions runner software on your own machines. You might use them when:
- You need specific hardware that GitHub doesn't provide
- You want to access internal networks or resources
- You need larger compute resources than GitHub-hosted runners offer
- You want to save on GitHub-hosted runner minutes

### 14. What are GitHub Actions environments and how do you use them for deployments?
**Answer:** Environments are named deployment targets that can have protection rules and secrets. You can set up approval requirements, wait timers, and limit which branches can deploy to them. You use them like this:
```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production
    steps:
      - run: ./deploy.sh
```

### 15. How do you handle conditional execution in GitHub Actions?
**Answer:** You can use the `if` keyword to conditionally run jobs or steps based on various conditions:
```yaml
steps:
  - name: Run only on main branch
    if: github.ref == 'refs/heads/main'
    run: echo "This runs only on the main branch"
  
  - name: Run only if previous step succeeded
    if: success()
    run: echo "Previous step was successful"
  
  - name: Run only if workflow was triggered by a PR
    if: github.event_name == 'pull_request'
    run: echo "This is a pull request"
```

## Scenario-Based Questions

### Scenario 1: Basic CI Pipeline for a JavaScript Project
**Question:** How would you set up a basic CI pipeline for a JavaScript project that runs tests on every push and pull request?

**Answer:**
```yaml
name: JavaScript CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '16'
          cache: 'npm'
      - name: Install dependencies
        run: npm ci
      - name: Run tests
        run: npm test
```

### Scenario 2: Building and Deploying a Frontend Application
**Question:** How would you create a workflow that builds a React application and deploys it to GitHub Pages?

**Answer:**
```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Set up Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '16'
          cache: 'npm'
      - name: Install dependencies
        run: npm ci
      - name: Build
        run: npm run build
      - name: Deploy to GitHub Pages
        uses: JamesIves/github-pages-deploy-action@v4
        with:
          folder: build
```

### Scenario 3: Multi-Environment Deployment Pipeline
**Question:** How would you create a deployment pipeline that deploys to staging automatically but requires approval for production?

**Answer:**
```yaml
name: Deploy Pipeline

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build application
        run: ./build.sh
      - name: Upload build artifacts
        uses: actions/upload-artifact@v3
        with:
          name: app-build
          path: dist/

  deploy-staging:
    needs: build
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - uses: actions/download-artifact@v3
        with:
          name: app-build
      - name: Deploy to staging
        run: ./deploy.sh staging

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment: production # This will require approval if set up in GitHub
    steps:
      - uses: actions/download-artifact@v3
        with:
          name: app-build
      - name: Deploy to production
        run: ./deploy.sh production
```

### Scenario 4: Testing Across Multiple Platforms and Versions
**Question:** How would you set up testing for a library that needs to work across multiple operating systems and language versions?

**Answer:**
```yaml
name: Cross-Platform Testing

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        python-version: ['3.8', '3.9', '3.10', '3.11']
    
    steps:
      - uses: actions/checkout@v3
      - name: Set up Python ${{ matrix.python-version }}
        uses: actions/setup-python@v4
        with:
          python-version: ${{ matrix.python-version }}
      - name: Install dependencies
        run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
      - name: Run tests
        run: pytest
```

### Scenario 5: Scheduled Security Scanning
**Question:** How would you set up a workflow that runs a security scan of your codebase on a weekly schedule?

**Answer:**
```yaml
name: Weekly Security Scan

on:
  schedule:
    - cron: '0 0 * * 0'  # Runs at midnight every Sunday

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Run security scan
        uses: github/codeql-action/analyze@v2
        with:
          languages: javascript, python
      - name: Send report by email
        if: always()  # Run even if the scan finds issues
        uses: dawidd6/action-send-mail@v3
        with:
          server_address: smtp.gmail.com
          server_port: 465
          username: ${{ secrets.EMAIL_USERNAME }}
          password: ${{ secrets.EMAIL_PASSWORD }}
          subject: Weekly Security Scan Results
          body: Security scan completed. See attached report.
          to: security-team@example.com
          from: GitHub Actions
          attachments: ./security-report.pdf
```

### Scenario 6: Handling Workflow Failures
**Question:** How would you modify a workflow to notify your team on Slack when a deployment fails?

**Answer:**
```yaml
name: Deployment Pipeline

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Deploy application
        id: deploy
        run: ./deploy.sh
        continue-on-error: true  # Continue to next step even if this fails
      
      - name: Notify success
        if: steps.deploy.outcome == 'success'
        uses: rtCamp/action-slack-notify@v2
        env:
          SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}
          SLACK_TITLE: Deployment Successful ✅
          SLACK_MESSAGE: 'The application was deployed successfully!'
      
      - name: Notify failure
        if: steps.deploy.outcome == 'failure'
        uses: rtCamp/action-slack-notify@v2
        env:
          SLACK_WEBHOOK: ${{ secrets.SLACK_WEBHOOK }}
          SLACK_COLOR: danger
          SLACK_TITLE: Deployment Failed ❌
          SLACK_MESSAGE: 'The deployment failed! Check the logs: ${{ github.server_url }}/${{ github.repository }}/actions/runs/${{ github.run_id }}'
      
      - name: Fail the workflow if deployment failed
        if: steps.deploy.outcome == 'failure'
        run: exit 1
```

These questions and scenarios cover a wide range of GitHub Actions capabilities from basic to advanced, using clear and simple language to explain the concepts.
