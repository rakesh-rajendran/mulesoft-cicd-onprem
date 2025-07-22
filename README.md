# MuleSoft CI/CD Pipeline Documentation - On-Premise Deployment

## Overview

This project implements a comprehensive CI/CD pipeline for MuleSoft applications using GitHub Actions and Maven for deployment to on-premise Mule runtime servers. The pipeline supports automated deployment to three environments: Development (DEV), Test (TEST), and Production (PROD) using Anypoint Runtime Manager (ARM).

## Project Structure

```
mulesoft-cicd-onprem/
├── .github/
│   └── workflows/
│       ├── dev-deploy.yml      # Development deployment workflow
│       ├── test-deploy.yml     # Test deployment workflow
│       └── prod-deploy.yml     # Production deployment workflow
├── pom.xml                     # Maven configuration with on-premise deployment profiles
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
- **On-Premise ARM Deployment**: Uses Mule Maven plugin for deployment to on-premise servers via ARM
- **Artifact Management**: Publishes artifacts to Anypoint Exchange for reuse

### Environment Profiles

The `pom.xml` defines three Maven profiles for on-premise deployment:

| Profile | Environment | Application Name | On-Premise Target | ARM Environment |
|---------|------------|------------------|-------------------|----------------|
| `dev` | DEV | `mulesoft-cicd-onprem-dev` | `mac-dev` | DEV |
| `test` | TEST | `mulesoft-cicd-onprem-test` | `mac-test` | TEST |
| `prod` | PROD | `mulesoft-cicd-onprem` | `mac-prod` | PROD |

### Key Properties

- **Runtime Version**: Mule 4.9.7
- **Java Version**: 17
- **Deployment Method**: ARM (Anypoint Runtime Manager)
- **Target Type**: Server (on-premise)
- **Server Targets**: 
  - `mac-dev` (Development server)
  - `mac-test` (Test server)
  - `mac-prod` (Production server)

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
   - Deploys to on-premise DEV server (`mac-dev`)
   - Uses environment protection rules

### 2. Test Deployment (`test-deploy.yml`)

**Trigger**: Pull Request to `test` branch

**Workflow Steps**:

1. **Smart Artifact Resolution**:
   - Attempts to download latest artifact from Exchange
   - Falls back to rebuilding if download fails
   - Extracts group ID dynamically from POM

2. **Deploy to TEST**:
   - Deploys to on-premise TEST server (`mac-test`)
   - Uses PR-specific SHA for deployment tracking

### 3. Production Deployment (`prod-deploy.yml`)

**Trigger**: Pull Request to `main` branch

**Workflow Steps**:

1. **Artifact Resolution with Validation**:
   - Attempts to download latest artifact from Exchange
   - Rebuilds if download fails
   - Validates artifact existence before deployment

2. **Deploy to PROD**:
   - Deploys to on-premise PROD server (`mac-prod`)
   - Includes additional validation steps
   - Uses environment protection rules

## Required GitHub Secrets

The following secrets must be configured in your GitHub repository:

### Connected App Credentials
- **`CONNECTED_APP_CLIENT_ID`**: Client ID of the Anypoint Platform Connected App
- **`CONNECTED_APP_CLIENT_SECRET`**: Client Secret of the Anypoint Platform Connected App

### Anypoint Platform Credentials
- **`ANYPOINT_USERNAME`**: Username for Anypoint Platform authentication
- **`ANYPOINT_PASSWORD`**: Password for Anypoint Platform authentication

### Setting up Connected App

1. Log in to Anypoint Platform
2. Navigate to Access Management → Connected Apps
3. Create a new Connected App with the following scopes:
   - `Design Center Developer`
   - `Exchange Contributor`
   - `Runtime Manager`

## On-Premise Server Configuration

### Prerequisites

Before deploying to on-premise servers, ensure:

1. **Server Registration**: Your on-premise Mule runtime servers must be registered in Anypoint Runtime Manager:
   - `mac-dev` - Development server
   - `mac-test` - Test server
   - `mac-prod` - Production server

2. **Server Requirements**:
   - Mule Runtime 4.9.7 installed and running
   - Servers registered and visible in ARM
   - Proper network connectivity to Anypoint Platform

3. **Server Status**: Verify servers are online and healthy in Runtime Manager

### Server Registration Steps

1. Download and install Mule Runtime 4.9.7 on each server
2. Register each server with Anypoint Platform using the registration token
3. Verify server connectivity in Runtime Manager console
4. Configure server groups if needed for load balancing

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
3. Application deployed to `mac-dev` server as `mulesoft-cicd-onprem-dev`

### Test Deployment
1. Create Pull Request to `test` branch
2. Automatic deployment to TEST environment
3. Uses latest artifact from Exchange or rebuilds if needed
4. Application deployed to `mac-test` server as `mulesoft-cicd-onprem-test`

### Production Deployment
1. Create Pull Request to `main` branch
2. Manual approval required (if protection rules configured)
3. Automatic deployment to PROD environment
4. Application deployed to `mac-prod` server as `mulesoft-cicd-onprem`

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
   - Check on-premise server availability in ARM
   - Validate environment-specific configurations
   - Ensure servers are registered and online

3. **Exchange Publishing Issues**:
   - Ensure proper organization ID in `pom.xml`
   - Verify Exchange permissions for Connected App
   - Check artifact naming conventions

4. **On-Premise Server Issues**:
   - Verify server registration in ARM
   - Check server connectivity to Anypoint Platform
   - Ensure Mule runtime version compatibility (4.9.7)
   - Validate server resource availability

### Logs and Monitoring

- Monitor deployments in Anypoint Runtime Manager
- Check GitHub Actions logs for detailed error information
- Use on-premise server logs for runtime issues
- Monitor server health and resource utilization in ARM

## Security Considerations

- Never commit credentials to version control
- Use GitHub Secrets for all sensitive information
- Regularly rotate Connected App credentials and platform passwords
- Implement proper environment protection rules
- Use least privilege principle for Connected App scopes
- Secure on-premise server access and network connectivity

## Performance Optimization

- Utilize Maven dependency caching
- Implement artifact reuse between environments
- Monitor on-premise server resource utilization
- Consider server clustering for high availability
- Optimize application memory and CPU usage

## Maintenance

- Regularly update Mule runtime versions across all servers
- Keep Maven plugin versions current
- Review and update dependency versions
- Monitor security advisories for third-party components
- Maintain server health and perform regular maintenance
- Keep server registration tokens current

## On-Premise vs CloudHub Differences

This deployment model differs from CloudHub in several key ways:

1. **Target Infrastructure**: Deploys to customer-managed servers instead of MuleSoft-managed infrastructure
2. **Resource Management**: Server resources are managed by the customer
3. **Scaling**: Manual scaling and load balancing configuration
4. **Monitoring**: Requires additional monitoring setup for server health
5. **Maintenance**: Customer responsible for server maintenance and updates