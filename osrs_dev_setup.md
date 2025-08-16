# OSRS Development Environment Setup Guide for Windows 11

## Table of Contents
1. [Introduction & Prerequisites](#introduction--prerequisites)
2. [Core System Dependencies](#core-system-dependencies)
3. [Development Environment Configuration](#development-environment-configuration)
4. [RuneLite Development Setup](#runelite-development-setup)
5. [Alternative Development Paths](#alternative-development-paths)
6. [Troubleshooting & Optimization](#troubleshooting--optimization)
7. [Development Best Practices](#development-best-practices)
8. [Next Steps & Resources](#next-steps--resources)

## Introduction & Prerequisites

This guide provides a comprehensive setup process for developing Old School RuneScape (OSRS) modifications on Windows 11 using Cursor IDE. We focus on RuneLite plugin development as the primary approach, with coverage of private server development as an alternative path.

### System Requirements
- **OS**: Windows 11 (build 22000 or later)
- **RAM**: 8GB minimum, 16GB recommended
- **Storage**: 10GB free space for development tools and dependencies
- **Network**: Stable internet connection for package downloads

### Development Philosophy
This guide emphasizes terminal-driven workflows that leverage both Windows Terminal and Cursor's integrated terminal. The setup enables Cursor's AI agent to assist with dependency management, configuration issues, and development tasks throughout the entire workflow.

## Core System Dependencies

### 2.1 Java Development Kit (JDK) 11

**Why JDK 11**: RuneLite specifically requires Java 11 with Eclipse Temurin distribution for optimal compatibility and performance.

#### Installation via Windows Package Manager (Recommended)

1. **Install Windows Package Manager** (if not already present):
   ```powershell
   # Open PowerShell as Administrator
   # winget should be available on Windows 11 by default
   winget --version
   ```

2. **Install Eclipse Temurin JDK 11**:
   ```powershell
   winget install EclipseAdoptium.Temurin.11.JDK
   ```

3. **Verify Installation**:
   ```powershell
   java -version
   javac -version
   ```
   Expected output should show Java 11.x.x with "Eclipse Temurin" in the description.

#### Manual Installation Alternative

If winget installation fails:

1. Download Eclipse Temurin 11 from [adoptium.net](https://adoptium.net/temurin/releases/?version=11)
2. Select **Windows x64 JDK .msi installer**
3. Run installer with default options
4. Verify installation using commands above

#### Environment Variable Configuration

Ensure `JAVA_HOME` is properly set:

```powershell
# Check current JAVA_HOME
echo $env:JAVA_HOME

# If not set, add to system environment variables
# Navigate to: System Properties > Environment Variables > System Variables
# Add: JAVA_HOME = C:\Program Files\Eclipse Adoptium\jdk-11.x.x-hotspot
# Add to PATH: %JAVA_HOME%\bin
```

### 2.2 Package Manager Setup

#### Windows Package Manager Configuration

Configure winget for enhanced package management:

```powershell
# Update winget sources
winget source update

# Configure winget settings for better output
winget settings --enable LocalManifestFiles
```

#### Chocolatey Installation (Alternative Package Manager)

For advanced package management capabilities:

```powershell
# Install Chocolatey
Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))

# Verify installation
choco --version
```

### 2.3 Version Control System

#### Git Installation

```powershell
# Via winget (recommended)
winget install Git.Git

# Via Chocolatey (alternative)
choco install git

# Verify installation
git --version
```

#### Git Configuration

```powershell
# Set global configuration
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"

# Configure line endings for Windows
git config --global core.autocrlf true

# Set default branch name
git config --global init.defaultBranch main
```

#### SSH Key Generation for GitHub

```powershell
# Generate SSH key
ssh-keygen -t ed25519 -C "your.email@example.com"

# Start SSH agent and add key
ssh-agent
ssh-add ~/.ssh/id_ed25519

# Copy public key to clipboard
Get-Content ~/.ssh/id_ed25519.pub | Set-Clipboard
```

## Development Environment Configuration

### 3.1 Cursor IDE Setup

#### Installation

```powershell
# Download and install Cursor from https://cursor.so/
# Or via winget if available
winget search cursor
```

#### Essential Extensions for Java Development

Install these extensions through Cursor's extension marketplace:

1. **Extension Pack for Java** (Microsoft)
   - Includes Language Support, Debugging, Testing, Maven, Project Management
2. **Lombok Annotations Support for VS Code** (Gabriel Basilio Brito)
3. **Maven for Java** (Microsoft)
4. **Git Extension Pack** (Don Jayamanne)
5. **GitLens** (GitKraken)

#### Terminal Integration Configuration

Configure Cursor's integrated terminal:

1. Open Cursor Settings (`Ctrl+,`)
2. Navigate to **Terminal > Integrated**
3. Set **Shell: Windows** to `PowerShell`
4. Configure **Font Family** for better readability
5. Enable **Copy on Selection** for efficient workflow

#### AI Agent Optimization

Configure Cursor's AI for development workflow:

1. **Enable Composer Mode** for multi-file editing
2. **Configure AI Model Preferences** for code assistance
3. **Set up Workspace-Specific AI Rules** for Java development

### 3.2 Build Tools

#### Maven Installation

```powershell
# Via winget
winget install Apache.Maven

# Via Chocolatey
choco install maven

# Verify installation
mvn --version
```

#### Gradle Setup (For Private Server Development)

```powershell
# Via winget
winget install Gradle.Gradle

# Verify installation
gradle --version
```

#### Build Tool Path Verification

```powershell
# Check all tools are in PATH
Get-Command java, javac, git, mvn, gradle | Select-Object Name, Source
```

### 3.3 Terminal Environment Enhancement

#### Windows Terminal Configuration

Install and configure Windows Terminal:

```powershell
# Install Windows Terminal (if not present)
winget install Microsoft.WindowsTerminal

# Launch Windows Terminal
wt
```

#### PowerShell Profile Customization

Create a PowerShell profile for development aliases:

```powershell
# Check if profile exists
Test-Path $PROFILE

# Create profile if it doesn't exist
if (!(Test-Path $PROFILE)) {
    New-Item -Type File -Path $PROFILE -Force
}

# Edit profile
notepad $PROFILE
```

Add these aliases to your PowerShell profile:

```powershell
# Development aliases
function rl-build { mvn clean install -DskipTests }
function rl-run { mvn exec:java -Dexec.mainClass="net.runelite.client.RuneLite" -Dexec.args="--debug --developer-mode" }
function git-status { git status --short }
function git-log-graph { git log --oneline --graph --decorate --all }

# Set aliases
Set-Alias gs git-status
Set-Alias gl git-log-graph
Set-Alias rlb rl-build
Set-Alias rlr rl-run

# Java version management
function java-version { java -version; javac -version }
Set-Alias jv java-version

Write-Host "OSRS Development Environment Loaded" -ForegroundColor Green
```

## RuneLite Development Setup

### 4.1 Repository Management

#### Clone RuneLite Repository

```powershell
# Create development directory
mkdir C:\Dev\OSRS
cd C:\Dev\OSRS

# Clone RuneLite repository
git clone https://github.com/runelite/runelite.git
cd runelite

# Verify repository structure
ls
```

#### Plugin Hub Template Setup

```powershell
# Navigate to plugin development directory
cd C:\Dev\OSRS

# Clone example plugin template
git clone https://github.com/runelite/example-plugin.git my-first-plugin
cd my-first-plugin

# Initialize as new repository
rm -rf .git
git init
git add .
git commit -m "Initial plugin template"
```

### 4.2 Build Configuration in Cursor

#### Open Project in Cursor

```powershell
# Open RuneLite in Cursor
cd C:\Dev\OSRS\runelite
cursor .

# Open plugin project in new window
cd C:\Dev\OSRS\my-first-plugin
cursor .
```

#### Configure Build Tasks

Create `.vscode/tasks.json` in your project root:

```json
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "Maven: Clean Install",
            "type": "shell",
            "command": "mvn",
            "args": ["clean", "install", "-DskipTests"],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "problemMatcher": "$maven-problems",
            "detail": "Clean and install Maven project"
        },
        {
            "label": "RuneLite: Build and Run",
            "type": "shell",
            "command": "mvn",
            "args": [
                "exec:java",
                "-Dexec.mainClass=net.runelite.client.RuneLite",
                "-Dexec.args=--debug --developer-mode"
            ],
            "group": "test",
            "dependsOn": "Maven: Clean Install",
            "detail": "Build and run RuneLite with debug mode"
        }
    ]
}
```

#### Lombok Plugin Integration

Ensure Lombok is properly configured:

1. Install **Lombok Annotations Support** extension in Cursor
2. Add to project `.vscode/settings.json`:

```json
{
    "java.configuration.updateBuildConfiguration": "automatic",
    "java.compile.nullAnalysis.mode": "automatic",
    "lombok.enabled": true,
    "java.jdt.ls.vmargs": "-javaagent:lombok.jar"
}
```

### 4.3 Development Workflow

#### Initial Build Process

```powershell
# Navigate to RuneLite project
cd C:\Dev\OSRS\runelite

# First-time build (downloads dependencies)
mvn clean install -DskipTests

# Run RuneLite client
mvn exec:java -Dexec.mainClass="net.runelite.client.RuneLite" -Dexec.args="--debug --developer-mode"
```

#### Plugin Development Cycle

For developing plugins:

```powershell
# Navigate to plugin project
cd C:\Dev\OSRS\my-first-plugin

# Build plugin
mvn clean install

# Test plugin (requires RuneLite client running)
# Place compiled JAR in RuneLite plugins directory
```

#### Debugging Setup

Configure launch configuration in `.vscode/launch.json`:

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "java",
            "name": "RuneLite Debug",
            "request": "launch",
            "mainClass": "net.runelite.client.RuneLite",
            "projectName": "runelite-client",
            "args": ["--debug", "--developer-mode"],
            "vmArgs": ["-ea"],
            "console": "integratedTerminal"
        }
    ]
}
```

## Alternative Development Paths

### 5.1 Private Server Development

#### Framework Selection

For comprehensive server modifications, consider these frameworks:

**RSBox Framework** (Recommended for beginners):
```powershell
# Clone RSBox
git clone https://github.com/vanicosrs/rsbox.git
cd rsbox

# Run setup wizard
java -Xmx1G -Xms1G -jar rsbox-server-0.1.jar --setup

# Start server
java -Xmx1G -Xms1G -jar rsbox-server-0.1.jar
```

**Apollo Server Suite** (Advanced):
```powershell
# Clone Apollo
git clone https://github.com/apollo-rsps/apollo.git
cd apollo

# Build with Gradle
gradle build

# Configure and run
# (Requires cache files and RSA key setup)
```

#### Additional Dependencies for Private Servers

```powershell
# Database setup (optional, for account management)
# MongoDB via Docker
winget install Docker.DockerDesktop

# After Docker installation
docker run --name mongodb -p 27017:27017 -d mongo

# MySQL alternative
winget install Oracle.MySQL
```

### 5.2 Hybrid Development Approaches

For projects requiring both client and server modifications:

1. **Client-side**: Use RuneLite plugin development
2. **Server-side**: Implement private server with API endpoints
3. **Communication**: HTTP/WebSocket APIs between client plugins and server

## Troubleshooting & Optimization

### 6.1 Common Setup Issues

#### Java Path Problems

```powershell
# Diagnose Java issues
where java
echo $env:JAVA_HOME
echo $env:PATH

# Fix PATH issues
$env:PATH += ";C:\Program Files\Eclipse Adoptium\jdk-11.x.x-hotspot\bin"
```

#### Build Failures

```powershell
# Clear Maven cache
mvn dependency:purge-local-repository

# Force update dependencies
mvn clean install -U

# Check for conflicting Java versions
java -version
mvn --version
```

#### Cursor Configuration Issues

1. **Java Extension Not Working**: Restart Cursor after JDK installation
2. **Lombok Errors**: Ensure annotation processing is enabled
3. **Maven Integration Issues**: Check Java extension pack is installed

### 6.2 Performance Optimization

#### JVM Tuning for Development

Add to your PowerShell profile:

```powershell
# Set Maven options for better performance
$env:MAVEN_OPTS = "-Xmx4g -Xms1g -XX:+UseG1GC"

# Set Java options for RuneLite development
$env:JAVA_OPTS = "-Xmx2g -Xms1g"
```

#### Build Acceleration

```powershell
# Enable parallel builds
mvn clean install -T 4 -DskipTests

# Use offline mode for subsequent builds
mvn compile -o
```

## Development Best Practices

### 7.1 Project Organization

#### Recommended Directory Structure

```
C:\Dev\OSRS\
├── runelite\              # Main RuneLite repository
├── my-plugins\            # Your plugin projects
│   ├── plugin-one\
│   ├── plugin-two\
│   └── shared-utils\
├── private-servers\       # Server development
│   ├── rsbox\
│   └── custom-server\
└── resources\            # Shared resources
    ├── caches\
    ├── documentation\
    └── scripts\
```

#### Version Control Workflows

```powershell
# Create feature branch for new plugin
git checkout -b feature/new-minigame-plugin

# Commit with conventional commits
git commit -m "feat: add new minigame overlay system"

# Push and create pull request
git push origin feature/new-minigame-plugin
```

### 7.2 AI-Assisted Development with Cursor

#### Optimizing Cursor's AI for OSRS Development

1. **Context-Aware Prompts**: Include RuneLite API references in your prompts
2. **Code Review**: Use AI to review plugin code for RuneLite best practices
3. **Documentation Generation**: Generate JavaDoc comments for plugin methods
4. **Debugging Assistance**: Ask AI to analyze error logs and suggest fixes

#### Example AI Prompts for OSRS Development

```
"Create a RuneLite plugin that overlays player health bars above NPCs, following RuneLite's overlay system patterns"

"Debug this Maven build error in my RuneLite plugin project"

"Explain how to use RuneLite's event system to detect when a player enters combat"

"Generate unit tests for this RuneLite plugin configuration class"
```

## Next Steps & Resources

### 8.1 Community Resources

#### Essential Links

- **RuneLite Developer Discord**: [discord.gg/runelite](https://discord.gg/runelite)
- **Plugin Hub**: [runelite.net/plugin-hub](https://runelite.net/plugin-hub)
- **RuneLite API Documentation**: [static.runelite.net/runelite-api/apidocs/](https://static.runelite.net/runelite-api/apidocs/)
- **OSRS Wiki Developer Tools**: [oldschool.runescape.wiki](https://oldschool.runescape.wiki)

#### Development Communities

- **Rune-Server**: Private server development community
- **OSRSBox**: Technical OSRS development blog
- **GitHub**: Search for "runelite plugin" for examples

### 8.2 Advanced Topics

#### Plugin Publishing Workflow

1. **Development**: Create plugin using template
2. **Testing**: Test with local RuneLite build
3. **Documentation**: Create README and usage guides
4. **Submission**: Submit to Plugin Hub via pull request
5. **Review**: Address feedback from RuneLite developers

#### Code Quality Tools

```powershell
# Install SpotBugs for static analysis
mvn com.github.spotbugs:spotbugs-maven-plugin:spotbugs

# Run Checkstyle for code formatting
mvn checkstyle:check

# Generate test coverage reports
mvn jacoco:report
```

#### Continuous Integration Setup

Create `.github/workflows/build.yml` for automated testing:

```yaml
name: Build and Test
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Set up JDK 11
      uses: actions/setup-java@v3
      with:
        java-version: '11'
        distribution: 'temurin'
    - name: Build with Maven
      run: mvn clean compile test
```

---

## Conclusion

This comprehensive setup guide provides everything needed to begin OSRS modification development on Windows 11. The terminal-centric approach ensures compatibility with Cursor's AI assistant while maintaining professional development standards. Start with RuneLite plugin development for immediate results, then explore private server development as your expertise grows.

Remember to leverage Cursor's AI capabilities throughout your development journey – from initial setup troubleshooting to advanced plugin architecture decisions. The combination of proper tooling and AI assistance creates an optimal environment for OSRS development innovation.