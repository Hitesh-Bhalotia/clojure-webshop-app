# Clojure Webshop Application - Jenkins Pipeline

This Jenkins pipeline automates the CI/CD process for a Clojure webshop application with specialized security scanning, code quality analysis, and containerized deployment.

## 📄 Project Attribution

**Important Note**: This project is a fork of the original Clojure Webshop application created by [tbsschroeder](https://github.com/tbsschroeder/clojure-webshop-app).

- **Original Repository**: https://github.com/tbsschroeder/clojure-webshop-app
- **Fork Repository**: https://github.com/Hitesh-Bhalotia/clojure-webshop-app.git
- **My Contribution**: The Jenkins pipeline configuration and CI/CD automation setup documented in this README

All credit for the Clojure webshop application code (frontend, backend, and core functionality) goes to the original author. This documentation focuses specifically on the Jenkins pipeline implementation and DevOps automation added to the forked repository.

## 🏗️ Pipeline Overview

This specialized declarative Jenkins pipeline performs the following operations:
- Source code checkout from GitHub
- Clojure-specific security scanning with clj-holmes
- SonarQube code quality analysis
- OWASP dependency vulnerability scanning
- Containerized deployment using Docker Compose

## 📋 Prerequisites

### Jenkins Requirements
- Jenkins server with pipeline plugin support
- Required Jenkins plugins (detailed in Plugin Installation section)
- Jenkins credentials configured for external services

### Tool Configurations
Configure the following tools in Jenkins Global Tool Configuration:

#### JDK Configuration
- **Name**: `jdk`
- **Type**: JDK installation
- **Installation**: Eclipse Temurin installer plugin (automatic)
- **Version**: Java 8 or higher (Java 11+ recommended for Clojure)

#### SonarQube Scanner Configuration
- **Name**: `sonar-scanner`
- **Type**: SonarQube Scanner installation
- **Installation**: Automatic installation
- **Version**: Latest stable version

#### Docker Configuration
- **Name**: `docker`
- **Type**: Docker installation
- **Installation**: Automatic installation
- **Integration**: Docker Pipeline plugin integration

#### OWASP Dependency Check
- **Name**: `DC`
- **Type**: Dependency-Check installation
- **Installation**: Automatic installer

### System Dependencies
Ensure the following tools are installed on Jenkins agents:
- **Clojure CLI**: For Clojure project management
- **clj-holmes**: Clojure-specific security scanner
- **Docker**: For containerized deployment
- **Docker Compose**: For multi-container orchestration
- **Git**: For source code management

## 🚀 Pipeline Stages

### 1. Git Checkout
```groovy
git 'https://github.com/Hitesh-Bhalotia/clojure-webshop-app.git'
```
- **Purpose**: Clones the source code from the repository
- **Repository**: `https://github.com/Hitesh-Bhalotia/clojure-webshop-app.git`
- **Branch**: Default branch (typically `main` or `master`)
- **Method**: Simple Git checkout for Clojure project

### 2. CLJ-HOLMES Security Scan
```groovy
sh "clj-holmes fetch-rules"
sh "clj-holmes scan -p ./"
```
- **Purpose**: Performs Clojure-specific security vulnerability scanning
- **Tool**: clj-holmes - specialized security scanner for Clojure applications
- **Process**:
  - **Fetch Rules**: Downloads latest security rules and patterns
  - **Scan**: Analyzes entire project directory for Clojure-specific vulnerabilities
- **Scope**: All Clojure source files, dependencies, and configurations
- **Detects**:
  - Unsafe Clojure expressions
  - Security anti-patterns
  - Vulnerable dependency usage
  - Code injection vulnerabilities

### 3. SonarQube Analysis
```groovy
withSonarQubeEnv('sonar') {
    sh '''$SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=WEBSHOP \
    -Dsonar.java.binaries=. \
    -Dsonar.projectKey=WEBSHOP '''
}
```
- **Purpose**: Performs static code analysis for quality and security
- **Tool**: SonarQube Scanner
- **Project**: WEBSHOP
- **Configuration**:
  - **Project Name**: WEBSHOP
  - **Project Key**: WEBSHOP
  - **Java Binaries**: Current directory (for JVM bytecode analysis)
- **Analysis Coverage**:
  - Code quality metrics
  - Security vulnerabilities
  - Code smells and maintainability issues
  - Clojure-specific code patterns

### 4. OWASP Dependency Check
```groovy
dependencyCheck additionalArguments: ' --scan ./ ', odcInstallation: 'DC'
dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
```
- **Purpose**: Scans project dependencies for known vulnerabilities
- **Tool**: OWASP Dependency Check
- **Scope**: Entire project directory (`./`)
- **Coverage**: 
  - Clojure/Leiningen dependencies (project.clj)
  - Java JAR dependencies
  - Transitive dependencies
- **Output**: Generates `dependency-check-report.xml`
- **Integration**: Results published to Jenkins dashboard

### 5. Build & Deploy
```groovy
sh "docker-compose up -d"
```
- **Purpose**: Builds and deploys the application using Docker Compose
- **Command**: Runs in detached mode (`-d`) for background execution
- **Requirement**: `docker-compose.yml` must exist in project root
- **Process**:
  - Builds Docker images for Clojure application
  - Creates and starts containers
  - Sets up networking between services
  - Starts the webshop application

## 📁 Expected Project Structure

```
clojure-webshop-app/
├── src/                  # Clojure source code
│   └── clojure/
│       └── webshop/
├── resources/            # Application resources
├── test/                 # Clojure test files
├── project.clj           # Leiningen project configuration
├── deps.edn             # Clojure CLI dependencies (if used)
├── docker-compose.yml    # Docker Compose configuration (REQUIRED)
├── Dockerfile           # Docker image build instructions
└── [other project files]
```

## 🛠️ Plugin Installation

Install the required plugins via Jenkins Plugin Manager:

### Core Plugins
- **JDK Tool**: For Java Development Kit management
- **Pipeline Plugin**: For declarative pipeline support
- **Git Plugin**: For source code management

### Security & Quality Plugins
- **OWASP Dependency-Check Plugin**: Vulnerability scanning
- **SonarQube Scanner Plugin**: Code quality analysis

### Deployment Plugins
- **Docker Plugin**: Docker integration
- **Docker Pipeline Plugin**: Docker pipeline steps
- **Docker Build Step Plugin**: Enhanced Docker build capabilities

### Installation Steps
1. Navigate to Jenkins → Manage Jenkins → Manage Plugins
2. Go to "Available" tab
3. Search and install each plugin listed above
4. Restart Jenkins when prompted


## 📊 Security Scanning Reports

### clj-holmes Scan Results
- **Output**: Jenkins console logs
- **Coverage**: Clojure-specific security issues
- **Detects**:
  - Use of dangerous functions (`eval`, `read-string` without validation)
  - SQL injection vulnerabilities
  - Path traversal issues
  - Unsafe serialization/deserialization
  - Hard-coded secrets

### OWASP Dependency Check
- **Report Location**: `dependency-check-report.xml`
- **Jenkins Integration**: Published with trend analysis
- **Coverage**: All Clojure/Java dependencies
- **Information**:
  - Known vulnerabilities (CVE)
  - CVSS scores
  - Affected components
  - Remediation suggestions

### SonarQube Analysis
- **Integration**: Real-time results in SonarQube dashboard
- **Metrics**: Code coverage, bugs, vulnerabilities, maintainability
- **Clojure Support**: Basic support for Clojure syntax and patterns
- **Quality Gates**: Configurable pass/fail criteria

## 🚀 Deployment Process

### What Happens During Deployment
1. **Image Building**: Builds Clojure application Docker image
2. **Dependency Resolution**: Downloads and caches Clojure dependencies
3. **Compilation**: Compiles Clojure code to JVM bytecode
4. **Service Creation**: Creates containers based on docker-compose.yml
5. **Network Setup**: Establishes container networking
6. **Application Startup**: Starts the Clojure webshop application

## 🔄 Version History

- **v1.0**: Initial Clojure-specific pipeline
  - Clojure security scanning with clj-holmes
  - SonarQube integration
  - OWASP dependency checking
  - Docker Compose deployment

---

**Note**: This pipeline is specifically optimized for Clojure applications. Ensure your project structure, dependency management, and Docker configuration match the requirements outlined in this documentation.
