# GitHub Actions Setup for Databricks CI/CD

This repository includes a comprehensive GitHub Actions workflow that automatically tests, validates, and deploys your Databricks Asset Bundle (DAB) project.

## Required GitHub Secrets

To use the CI/CD pipeline, you need to configure the following secrets in your GitHub repository:

### Repository Secrets

Go to **Settings** → **Secrets and variables** → **Actions** → **New repository secret**

1. **`DATABRICKS_HOST`**
   - **Description**: Your Databricks workspace URL
   - **Example**: `https://adb-984752964297111.11.azuredatabricks.net/`
   - **Note**: Include the trailing slash

2. **`DATABRICKS_TOKEN`**
   - **Description**: Personal Access Token for Databricks authentication
   - **How to create**:
     1. Log into your Databricks workspace
     2. Go to **Settings** → **Developer** → **Access tokens**
     3. Click **Generate new token**
     4. Give it a meaningful name (e.g., "GitHub Actions CI/CD")
     5. Set appropriate expiration (recommend 90-180 days)
     6. Copy the token value (you won't see it again!)
   - **Security**: This token provides full access to your workspace - keep it secure!

## Workflow Overview

The CI/CD pipeline includes four main jobs and supports both **automatic** and **parameter-controlled** deployments:

### **Automatic Triggers**
- **Push to `develop`** → Deploy to Dev
- **Push to `main`** → Deploy to Production (with approval)
- **Pull Requests** → Run tests and validation only

### **Manual Deployments with Parameters**
Go to **Actions** → **Databricks CI/CD Pipeline** → **Run workflow** to deploy with custom parameters:

#### Available Parameters:
1. **Target Environment** (required):
   - `dev` - Deploy to development environment only
   - `prod` - Deploy to production environment only  
   - `both` - Deploy to both dev and prod sequentially

2. **Run Tests** (optional, default: true):
   - Whether to run the test suite before deployment

3. **Run Job After Deploy** (optional, default: false):
   - Whether to trigger the Databricks job after successful deployment

### **Jobs Breakdown**

### 1. **Test** (`test`)
- Runs on all automatic triggers (push/PR) and manual runs (if enabled)
- Sets up Python environment and installs dependencies
- Runs pytest test suite
- Can be skipped in manual deployments

### 2. **Validate** (`validate`)  
- Always runs on all triggers
- Installs Databricks CLI
- Validates bundle configuration using `databricks bundle validate`
- Must pass for deployments to proceed

### 3. **Deploy to Dev** (`deploy-dev`)
- **Automatic**: Runs on `develop` branch pushes or pull requests
- **Manual**: Runs when environment is `dev` or `both`
- Deploys bundle to the `dev` target environment
- Optionally runs job after deployment

### 4. **Deploy to Production** (`deploy-prod`)
- **Automatic**: Runs on `main` branch pushes
- **Manual**: Runs when environment is `prod` or `both`
- Requires manual approval (uses `production` environment)  
- Deploys bundle to the `prod` target environment
- Optionally runs job after deployment

## Using Parameter-Controlled Deployments

### How to Trigger Manual Deployments

1. Go to your repository on GitHub
2. Click the **Actions** tab
3. Select **Databricks CI/CD Pipeline** from the workflows list
4. Click **Run workflow** button (top right)
5. Choose your parameters:
   - **Branch**: Select the branch to deploy from
   - **Target environment**: Choose `dev`, `prod`, or `both`
   - **Run tests**: Toggle test execution
   - **Run job after deploy**: Toggle job execution after deployment

### Common Use Cases

#### 🚀 **Quick Dev Deployment (skip tests)**
- Environment: `dev`
- Run tests: `false`
- Run job after deploy: `true`
- *Use case*: Fast iteration during development

#### 🔄 **Full Production Deployment**
- Environment: `prod`  
- Run tests: `true`
- Run job after deploy: `false`
- *Use case*: Controlled production release with full validation

#### 📋 **Deploy to Both Environments**
- Environment: `both`
- Run tests: `true`
- Run job after deploy: `true`
- *Use case*: Promote from dev to prod in one workflow

#### 🧪 **Testing Only**
- Don't use manual deployment, just push to a feature branch
- *Use case*: Validate changes without deploying

### Benefits of Parameter Control

- **Flexibility**: Deploy to any environment from any branch
- **Speed**: Skip tests for rapid development cycles
- **Testing**: Control job execution after deployment
- **Safety**: Manual approval still required for production
- **Efficiency**: Deploy to multiple environments in one go

## Setting Up Production Environment Protection

For production deployments, it's recommended to set up environment protection rules:

1. Go to **Settings** → **Environments**
2. Click **New environment** and name it `production`
3. Add protection rules:
   - **Required reviewers**: Add team members who should approve production deployments
   - **Wait timer**: Optional delay before deployment
   - **Deployment branches**: Restrict to `main` branch only

## Databricks CLI Installation

The workflow automatically installs the Databricks CLI using the official installation script:

```bash
curl -fsSL https://raw.githubusercontent.com/databricks/setup-cli/main/install.sh | sh
```

This ensures you always get the latest stable version of the CLI with all necessary dependencies.

## Customizing the Workflow

### Running Jobs After Deployment

The workflow includes commented-out steps to run jobs after deployment. To enable:

1. Uncomment the job run steps in the workflow file
2. Replace `dab_test_job` with your actual job name from `resources/dab_test.job.yml`

### Adding Additional Tests

You can extend the test job to include:
- Code linting (flake8, black, pylint)
- Security scanning
- Code coverage reporting
- Integration tests

### Multiple Environments

To add additional environments (e.g., staging):

1. Add the environment to your `databricks.yml` file
2. Create a new job in the workflow file
3. Set appropriate triggers and branch conditions

## Troubleshooting

### Common Issues

1. **Authentication fails**
   - Verify `DATABRICKS_HOST` includes protocol and trailing slash
   - Check that `DATABRICKS_TOKEN` is valid and not expired
   - Ensure the token has necessary permissions

2. **Bundle validation fails**
   - Check your `databricks.yml` syntax
   - Verify all referenced files exist
   - Ensure workspace paths are accessible

3. **Deployment fails**
   - Check workspace permissions
   - Verify target environment configuration
   - Ensure no conflicting resources exist

### Getting Help

- Check the [Databricks Asset Bundles documentation](https://docs.databricks.com/dev-tools/bundles/index.html)
- Review [GitHub Actions documentation](https://docs.github.com/en/actions)
- Check workflow run logs in the **Actions** tab of your repository

## Security Best Practices

1. **Token Management**
   - Use tokens with minimal required permissions
   - Set reasonable expiration periods
   - Rotate tokens regularly
   - Never commit tokens to code

2. **Environment Protection**
   - Always require manual approval for production
   - Use deployment branches to restrict access
   - Consider using service principals for production

3. **Audit and Monitoring**
   - Review deployment logs regularly
   - Monitor workspace changes
   - Set up alerts for failed deployments
