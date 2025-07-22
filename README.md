# MuleSoft CI/CD Pipeline Documentation

## Overview

This project implements a comprehensive CI/CD pipeline for MuleSoft applications using GitHub Actions and Maven. The pipeline supports automated deployment to three environments: Development (DEV), Test (TEST), and Production (PROD).

## Project Structure

```
mulesoft-ci-cd/
├── .github/
│   └── workflows/
│       ├── dev-deploy.yml      # Development deployment workflow
│       ├── test-deploy.yml     # Test deployment workflow
│       └── prod-deploy.yml     # Production deployment workflow
├── pom.xml                     # Maven configuration with multi-environment profiles
├── src/
│   ├── main/
│   │   ├── mule/              # Mule application flows
│   │   └── resources/         # Application resources and configurations
│   └── test/
│       └── munit/             # MUnit test cases
└── .maven/
    └── settings.xml           # Maven settings for Anypoint Platform authentication
```

## Maven Configuration (pom.xml)

### Key Features

- **Multi-Environment Support**: Configured with profiles for DEV, TEST, and PROD environments
- **CloudHub 2.0 Deployment**: Uses the latest Mule Maven plugin for CloudHub 2.0 deployments
- **Artifact Management**: Publishes artifacts to Anypoint Exchange for reuse

### Environment Profiles

The `pom.xml` defines three Maven profiles:

| Profile | Environment | Application Name | CloudHub Environment |
|---------|------------|------------------|---------------------|
| `dev` | DEV | `mulesoft-ci-cd-dev` | DEV |
| `test` | TEST | `mulesoft-ci-cd-test` | TEST |
| `prod` | PROD | `mulesoft-ci-cd` | PROD |

### Key Properties

- **Runtime Version**: Mule 4.9.7
- **Java Version**: 17
- **CloudHub Target**: Cloudhub-US-East-2
- **vCores**: 0.1 (Development sizing)
- **Replicas**: 1

### Dependencies

- **HTTP Connector**: v1.10.3
- **Sockets Connector**: v1.2.5

## CI/CD Workflows

### 1. Development Deployment (`dev-deploy.yml`)

**Trigger**: Push to `dev` branch

**Workflow Steps**:

1. **Build Job**:
   - Checks out code
   - Caches Maven dependencies
   - Sets up JDK 1.8
   - Runs `mvn clean compile test package`
   - Stamps artifact with commit hash
   - Uploads artifact for reuse

2. **Publish Job**:
   - Downloads built artifact
   - Publishes to Anypoint Exchange
   - Uses Connected App credentials for authentication

3. **Deploy Job**:
   - Downloads artifact
   - Deploys to CloudHub DEV environment
   - Uses environment protection rules

### 2. Test Deployment (`test-deploy.yml`)

**Trigger**: Pull Request to `test` branch

**Workflow Steps**:

1. **Smart Artifact Resolution**:
   - Attempts to download latest artifact from Exchange
   - Falls back to rebuilding if download fails
   - Extracts group ID dynamically from POM

2. **Deploy to TEST**:
   - Deploys to CloudHub TEST environment
   - Uses PR-specific SHA for deployment tracking

### 3. Production Deployment (`prod-deploy.yml`)

**Trigger**: Pull Request to `main` branch

**Workflow Steps**:

1. **Artifact Resolution with Validation**:
   - Attempts to download latest artifact from Exchange
   - Rebuilds if download fails
   - Validates artifact existence before deployment

2. **Deploy to PROD**:
   - Deploys to CloudHub PROD environment
   - Includes additional validation steps
   - Uses environment protection rules

## Required GitHub Secrets

The following secrets must be configured in your GitHub repository:

### Connected App Credentials
- **`CONNECTED_APP_CLIENT_ID`**: Client ID of the Anypoint Platform Connected App
- **`CONNECTED_APP_CLIENT_SECRET`**: Client Secret of the Anypoint Platform Connected App

### Setting up Connected App

1. Log in to Anypoint Platform
2. Navigate to Access Management → Connected Apps
3. Create a new Connected App with the following scopes:
   - `Design Center Developer`
   - `Exchange Contributor`
   - `Cloudhub Application Developer`
   - `Runtime Manager`

## Environment Protection Rules

Configure GitHub environment protection rules for:

- **development**: For DEV deployments
- **test**: For TEST deployments  
- **production**: For PROD deployments (recommend required reviewers)

## Maven Settings

The pipeline uses `.maven/settings.xml` for Anypoint Platform authentication. Ensure this file contains:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 
          http://maven.apache.org/xsd/settings-1.0.0.xsd">
    
    <servers>
        <server>
            <id>Repository</id>
            <username>~~~Client~~~</username>
            <password>${client.id}~?~${client.secret}</password>
        </server>
        <server>
            <id>anypoint-exchange-v3</id>
            <username>~~~Client~~~</username>
            <password>${client.id}~?~${client.secret}</password>
        </server>
    </servers>
    
</settings>
```

## Deployment Process

### Development Deployment
1. Push code to `dev` branch
2. Automatic build, test, publish, and deploy
3. Application deployed as `mulesoft-ci-cd-dev`

### Test Deployment
1. Create Pull Request to `test` branch
2. Automatic deployment to TEST environment
3. Uses latest artifact from Exchange or rebuilds if needed
4. Application deployed as `mulesoft-ci-cd-test`

### Production Deployment
1. Create Pull Request to `main` branch
2. Manual approval required (if protection rules configured)
3. Automatic deployment to PROD environment
4. Application deployed as `mulesoft-ci-cd`

## Build Commands

### Local Development
```bash
# Clean and compile
mvn clean compile

# Run tests
mvn test

# Package application
mvn package

# Deploy to specific environment
mvn deploy -Denvironment=DEV -DmuleDeploy
```

### Testing
```bash
# Run MUnit tests
mvn test

# Skip tests during deployment
mvn deploy -DskipMunitTests
```

## Troubleshooting

### Common Issues

1. **Build Failures**:
   - Verify JDK version compatibility
   - Check Maven dependency resolution
   - Ensure MUnit tests pass

2. **Deployment Failures**:
   - Verify Connected App credentials
   - Check CloudHub resource availability
   - Validate environment-specific configurations

3. **Exchange Publishing Issues**:
   - Ensure proper organization ID in `pom.xml`
   - Verify Exchange permissions for Connected App
   - Check artifact naming conventions

### Logs and Monitoring

- Monitor deployments in Anypoint Runtime Manager
- Check GitHub Actions logs for detailed error information
- Use CloudHub application logs for runtime issues

## Security Considerations

- Never commit credentials to version control
- Use GitHub Secrets for all sensitive information
- Regularly rotate Connected App credentials
- Implement proper environment protection rules
- Use least privilege principle for Connected App scopes

## Performance Optimization

- Utilize Maven dependency caching
- Implement artifact reuse between environments
- Consider increasing vCores for production workloads
- Monitor CloudHub resource utilization

## Maintenance

- Regularly update Mule runtime versions
- Keep Maven plugin versions current
- Review and update dependency versions
- Monitor security advisories for third-party components