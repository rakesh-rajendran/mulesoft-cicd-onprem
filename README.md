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
└── README.md                   # This documentation file
```

## Architecture and Design Decisions

### Dynamic Configuration Management
- **Dynamic settings.xml Creation**: Maven settings are created dynamically in each workflow for better security and environment-specific configurations
- **No Static Configuration Files**: Eliminated static .maven/settings.xml to prevent credential exposure
- **Environment-Specific Parameters**: Each environment uses specific server targets and configurations

### Security and Authentication
- **Connected App Authentication**: Uses modern Connected App credentials for Anypoint Platform authentication
- **GitHub Secrets Management**: All sensitive information stored as GitHub repository secrets
- **No Username/Password**: Eliminated legacy username/password authentication in favor of Connected App credentials

## Maven Configuration (pom.xml)

### Key Features

- **Multi-Environment Support**: Configured with profiles for DEV, TEST, and PROD environments
- **On-Premise ARM Deployment**: Uses Mule Maven plugin for deployment to on-premise servers via ARM
- **Java 17 Compatibility**: Optimized for Java 17 runtime environment
- **Enhanced Connector Versions**: Uses updated connector versions for better stability

### Environment Profiles

The `pom.xml` defines three Maven profiles for on-premise deployment:

| Profile | Environment | Application Name | On-Premise Target | ARM Environment |
|---------|------------|------------------|-------------------|----------------|
| `dev` | DEV | `mulesoft-cicd-onprem-dev` | `mac-dev` | DEV |
| `test` | TEST | `mulesoft-cicd-onprem-test` | `mac-test` | TEST |
| `prod` | PROD | `mulesoft-cicd-onprem` | `mac-prod` | PROD |

### Key Properties

- **Runtime Version**: Mule 4.9.7
- **Java Version**: 17 (compatible with connector requirements)
- **Deployment Method**: ARM (Anypoint Runtime Manager)
- **Target Type**: Server (on-premise)
- **Server Targets**: 
  - `mac-dev` (Development server)
  - `mac-test` (Test server)
  - `mac-prod` (Production server)

### Dependencies

- **HTTP Connector**: v1.10.0 (Java 17 compatible)
- **Sockets Connector**: v1.2.4 (stable version)

## Branching Strategy

### Branch Structure

```
main (Production)
├── test (Test Environment)
├── dev (Development Environment)
└── feature/* (Feature branches)
```

### Branching Workflow

1. **Feature Development**:
   - Create feature branches from `dev`
   - Naming convention: `feature/[feature-name]`
   - Example: `feature/add-logging-enhancement`

2. **Development Integration**:
   - Merge feature branches into `dev` branch
   - Automatic deployment to DEV environment (`mac-dev`)
   - Integration testing in development environment

3. **Test Promotion**:
   - Create Pull Request from `dev` to `test`
   - Manual approval required
   - Automatic deployment to TEST environment (`mac-test`)
   - User acceptance testing

4. **Production Release**:
   - Create Pull Request from `test` to `main`
   - Manual approval with enhanced protection rules
   - Automatic deployment to PROD environment (`mac-prod`)
   - Production monitoring and validation

### Branch Protection Rules

- **`main` branch**: 
  - Requires pull request reviews
  - Requires status checks to pass
  - Requires branches to be up to date before merging
  - Restricts pushes to administrators only

- **`test` branch**:
  - Requires pull request reviews
  - Requires status checks to pass

- **`dev` branch**:
  - Allows direct pushes for rapid development
  - Automatic CI/CD triggers

## CI/CD Workflows

### 1. Development Deployment (`dev-deploy.yml`)

**Trigger**: Push to `dev` branch

**Workflow Steps**:

1. **Build Job**:
   - Checks out code
   - Caches Maven dependencies with optimized cache keys
   - Sets up JDK 1.8 for build compatibility
   - Runs `mvn clean compile test package`
   - Stamps artifact with commit hash for traceability
   - Uploads artifact for deployment reuse

2. **Deploy Job**:
   - Creates dynamic Maven settings.xml with authentication
   - Downloads built artifact
   - Validates artifact existence
   - Extracts version information from POM
   - Deploys to on-premise DEV server (`mac-dev`)
   - Generates comprehensive deployment report

**Key Features**:
- **Enhanced Error Handling**: Validates artifacts before deployment
- **Rich Reporting**: GitHub Step Summary with deployment details
- **Optimized Caching**: Specific cache keys for better performance
- **Security**: Dynamic credential management

### 2. Test Deployment (`test-deploy.yml`)

**Trigger**: Pull Request to `test` branch

**Workflow Steps**:

1. **Smart Artifact Resolution**:
   - Attempts to download latest artifact from Exchange
   - Falls back to rebuilding if download fails
   - Extracts group ID dynamically from POM
   - Validates artifact integrity

2. **Deploy to TEST**:
   - Creates dynamic Maven settings.xml
   - Deploys to on-premise TEST server (`mac-test`)
   - Uses PR-specific SHA for deployment tracking
   - Provides detailed deployment feedback

### 3. Production Deployment (`prod-deploy.yml`)

**Trigger**: Pull Request to `main` branch

**Workflow Steps**:

1. **Enhanced Artifact Resolution**:
   - Attempts to download latest artifact from Exchange
   - Rebuilds if download fails with comprehensive error handling
   - Validates artifact existence before deployment
   - Implements retry logic for robust operations

2. **Deploy to PROD**:
   - Creates dynamic Maven settings.xml with enhanced security
   - Deploys to on-premise PROD server (`mac-prod`)
   - Includes additional validation steps
   - Uses environment protection rules for safety

## Deployment Strategy

### Environment Progression

```
Development → Test → Production
    ↓           ↓        ↓
  mac-dev   mac-test  mac-prod
```

### Deployment Patterns

1. **Continuous Deployment (DEV)**:
   - Automatic deployment on every push to `dev`
   - Rapid feedback loop for developers
   - Integration testing environment

2. **Continuous Delivery (TEST)**:
   - Deployment triggered by Pull Request
   - Manual approval for promotion
   - User acceptance testing environment

3. **Controlled Release (PROD)**:
   - Deployment triggered by Pull Request to main
   - Enhanced approval process
   - Production monitoring and rollback capabilities

### Deployment Validation

Each deployment includes:
- **Artifact Validation**: Ensures JAR file exists and is valid
- **Version Tracking**: Extracts and reports application version
- **Environment Verification**: Confirms target environment configuration
- **Deployment Reporting**: Comprehensive success/failure reporting
- **GitHub Integration**: Rich step summaries and status reporting

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
   - `Runtime Manager`
   - `Cloudhub Application Developer` (for ARM access)

## On-Premise Server Configuration

### Prerequisites

Before deploying to on-premise servers, ensure:

1. **Server Registration**: Your on-premise Mule runtime servers must be registered in Anypoint Runtime Manager:
   - `mac-dev` - Development server
   - `mac-test` - Test server
   - `mac-prod` - Production server

2. **Server Requirements**:
   - **Mule Runtime**: 4.9.7 installed and running
   - **Java Version**: 17 (CRITICAL: Java 21 not supported by HTTP connector)
   - **Server Registration**: Registered and visible in ARM
   - **Network Connectivity**: Proper connectivity to Anypoint Platform

3. **Server Status**: Verify servers are online and healthy in Runtime Manager

### Java Version Compatibility

⚠️ **IMPORTANT**: Your on-premise servers must run Java 17, not Java 21.

**Issue**: The HTTP connector (v1.10.0) supports Java versions [1.8, 11, 17] but NOT Java 21.

**Solution**: Configure your on-premise servers to use Java 17:

1. **Install Java 17** on all servers (mac-dev, mac-test, mac-prod)
2. **Update Mule Configuration**:
   ```bash
   # Edit wrapper.conf in your Mule installation
   sudo nano $MULE_HOME/conf/wrapper.conf
   
   # Update Java command
   wrapper.java.command=/path/to/java17/bin/java
   ```
3. **Restart Mule Runtime** on all servers
4. **Verify Java Version**: Confirm Mule is using Java 17

### Server Registration Steps

1. Download and install Mule Runtime 4.9.7 on each server
2. Configure servers to use Java 17 (critical step)
3. Register each server with Anypoint Platform using the registration token
4. Verify server connectivity in Runtime Manager console
5. Configure server groups if needed for load balancing

## Environment Protection Rules

Configure GitHub environment protection rules for:

### Development Environment
- **Name**: `development`
- **Protection**: Basic protection rules
- **Deployment**: Automatic on push to dev branch

### Test Environment
- **Name**: `test`
- **Protection**: 
  - Required reviewers (recommended)
  - Deployment branches: test branch only
- **Deployment**: Manual approval required

### Production Environment
- **Name**: `production`
- **Protection**: 
  - Required reviewers (mandatory)
  - Deployment branches: main branch only
  - Additional approval delays
- **Deployment**: Enhanced manual approval process

## Maven Configuration Details

### Dynamic Settings.xml Template

Each workflow creates a dynamic `settings.xml` with:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.2.0">
  <servers>
    <server>
      <id>Repository</id>
      <username>~~~Client~~~</username>
      <password>${client.id}~?~${client.secret}</password>
    </server>
  </servers>
  
  <profiles>
    <profile>
      <id>mule-extra-repos</id>
      <activation><activeByDefault>true</activeByDefault></activation>
      <repositories>
        <repository>
          <id>mule-public</id>
          <url>https://repository.mulesoft.org/nexus/content/repositories/public</url>
        </repository>
        <repository>
          <id>mulesoft-releases</id>
          <url>https://repository.mulesoft.org/releases/</url>
        </repository>
      </repositories>
    </profile>
  </profiles>
</settings>
```

### Repository Fallback Strategy

Maven resolves dependencies from:
1. **Organization's Exchange** (primary)
2. **MuleSoft Public Repository** (fallback)
3. **MuleSoft Releases Repository** (fallback)

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

### Environment-Specific Deployment
```bash
# Deploy to DEV
mvn deploy -Denvironment=DEV -DmuleDeploy -Dclient.id=xxx -Dclient.secret=xxx

# Deploy to TEST
mvn deploy -Denvironment=TEST -DmuleDeploy -Dclient.id=xxx -Dclient.secret=xxx

# Deploy to PROD
mvn deploy -Denvironment=PROD -DmuleDeploy -Dclient.id=xxx -Dclient.secret=xxx
```

## Troubleshooting

### Common Issues

1. **Java Version Incompatibility**:
   - **Error**: `JavaVersionNotSupportedByExtensionException: Extension 'HTTP' does not support Java 21`
   - **Solution**: Configure on-premise servers to use Java 17
   - **Verification**: Check server Java version in Runtime Manager

2. **Authentication Failures**:
   - **Error**: `401 Unauthorized: Missing credentials`
   - **Solution**: Verify Connected App credentials in GitHub secrets
   - **Check**: Ensure Connected App has proper scopes

3. **Deployment Failures**:
   - **Error**: Server not found or offline
   - **Solution**: Verify server registration in ARM
   - **Check**: Server connectivity to Anypoint Platform

4. **Dependency Resolution Issues**:
   - **Error**: Artifact not found
   - **Solution**: Check connector version availability
   - **Fallback**: Use public MuleSoft repositories

5. **Build Failures**:
   - **Error**: Maven compilation errors
   - **Solution**: Verify JDK version compatibility
   - **Check**: Maven dependency versions

### Server Configuration Issues

1. **Java Version Mismatch**:
   ```bash
   # Check current Java version on server
   java -version
   
   # Update Mule wrapper.conf
   wrapper.java.command=/usr/lib/jvm/java-17-openjdk/bin/java
   
   # Restart Mule service
   sudo systemctl restart mule
   ```

2. **Server Registration Problems**:
   - Verify registration token is valid
   - Check network connectivity to Anypoint Platform
   - Ensure server appears in Runtime Manager

### Logs and Monitoring

- **GitHub Actions Logs**: Check workflow execution details
- **Anypoint Runtime Manager**: Monitor server health and deployments
- **Application Logs**: Review on-premise server logs for runtime issues
- **Deployment Reports**: Check GitHub Step Summary for deployment details

## Security Considerations

### Authentication Security
- **No Static Credentials**: All credentials managed through GitHub secrets
- **Connected App Authentication**: Modern, secure authentication method
- **Dynamic Configuration**: Settings.xml created dynamically per deployment

### Access Control
- **GitHub Environment Protection**: Controls deployment approvals
- **Branch Protection Rules**: Prevents unauthorized code changes
- **Scope-Limited Connected Apps**: Minimal required permissions

### Network Security
- **Secure Communication**: All communication over HTTPS
- **Server Registration**: Servers authenticated via registration tokens
- **Firewall Configuration**: Ensure proper network access for ARM communication

## Performance Optimization

### Build Performance
- **Maven Caching**: Optimized cache keys for faster builds
- **Parallel Execution**: Jobs run concurrently where possible
- **Artifact Reuse**: Built artifacts reused across deployment stages

### Deployment Performance
- **Smart Artifact Resolution**: Efficient artifact management
- **Connection Pooling**: Optimized Maven repository connections
- **Retry Logic**: Robust error handling and recovery

### Server Performance
- **Resource Monitoring**: Monitor server resource utilization
- **Load Balancing**: Consider server clustering for high availability
- **Application Optimization**: Memory and CPU usage optimization

## Maintenance and Updates

### Regular Maintenance Tasks

1. **Dependency Updates**:
   - Review and update connector versions quarterly
   - Test compatibility with new versions
   - Update Maven plugin versions

2. **Security Updates**:
   - Rotate Connected App credentials annually
   - Review and update GitHub secrets
   - Update server certificates as needed

3. **Server Maintenance**:
   - Keep Mule runtime versions current
   - Update Java versions when connector support expands
   - Monitor server health and performance

4. **Documentation Updates**:
   - Keep README current with configuration changes
   - Update deployment procedures
   - Maintain troubleshooting guides

### Version Upgrade Strategy

1. **Connector Upgrades**:
   - Test new versions in development first
   - Verify Java compatibility
   - Update all environments consistently

2. **Runtime Upgrades**:
   - Plan coordinated upgrades across all servers
   - Test thoroughly in non-production environments
   - Schedule maintenance windows for production

3. **Java Version Upgrades**:
   - Monitor connector support for newer Java versions
   - Plan systematic upgrades when support is available
   - Test application compatibility thoroughly

## Best Practices

### Development Workflow
- **Feature Branches**: Use feature branches for all development
- **Code Reviews**: Mandatory code reviews for all changes
- **Testing**: Comprehensive testing in development environment
- **Documentation**: Keep code and deployment documentation current

### Deployment Practices
- **Gradual Rollouts**: Deploy through environments progressively
- **Rollback Planning**: Always have rollback procedures ready
- **Monitoring**: Monitor deployments and application health
- **Communication**: Notify stakeholders of deployment schedules

### Security Practices
- **Secret Management**: Secure handling of all credentials
- **Access Control**: Principle of least privilege
- **Audit Trail**: Maintain deployment and access logs
- **Regular Reviews**: Periodic security assessments

## Support and Troubleshooting Contacts

### Technical Support
- **MuleSoft Support**: For runtime and connector issues
- **GitHub Support**: For CI/CD pipeline issues
- **Infrastructure Team**: For server configuration issues

### Documentation References
- **MuleSoft Documentation**: https://docs.mulesoft.com/
- **GitHub Actions**: https://docs.github.com/en/actions
- **Maven Plugin**: https://docs.mulesoft.com/mule-runtime/4.4/deploy-on-premises

This comprehensive guide covers all aspects of the on-premise deployment pipeline, from initial setup to ongoing maintenance. Follow these guidelines to ensure reliable, secure, and efficient deployments to your on-premise Mule runtime environment.