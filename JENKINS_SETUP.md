# Jenkins Setup Guide for Matrix API CI/CD Pipeline

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Jenkins Installation](#jenkins-installation)
3. [Plugin Installation](#plugin-installation)
4. [Tool Configuration](#tool-configuration)
5. [Credentials Setup](#credentials-setup)
6. [SonarQube Configuration](#sonarqube-configuration)
7. [Project Setup](#project-setup)
8. [GitHub Webhook Configuration](#github-webhook-configuration)

---

## Prerequisites

- Java JDK 11 or higher
- Jenkins 2.x or higher
- SonarQube Server
- GitHub account
- MyMavenRepo account

---

## Jenkins Installation

### Windows
1. Download Jenkins WAR file from https://www.jenkins.io/download/
2. Run: `java -jar jenkins.war --httpPort=8080`
3. Open browser: http://localhost:8080
4. Follow the setup wizard
5. Install suggested plugins

### Linux/Mac
```bash
# Download and install Jenkins
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt-get update
sudo apt-get install jenkins

# Start Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

---

## Plugin Installation

Go to `Manage Jenkins` > `Manage Plugins` > `Available` and install:

### Required Plugins
- [x] **Pipeline** - Core pipeline functionality
- [x] **Git Plugin** - Git SCM support
- [x] **Gradle Plugin** - Gradle build tool
- [x] **SonarQube Scanner** - Code quality analysis
- [x] **JaCoCo Plugin** - Code coverage
- [x] **HTML Publisher Plugin** - Publish HTML reports
- [x] **JUnit Plugin** - Test result publishing
- [x] **Email Extension Plugin** - Email notifications
- [x] **Slack Notification Plugin** - Slack notifications
- [x] **Pipeline: Stage View Plugin** - Pipeline visualization
- [x] **Blue Ocean** (Optional) - Modern UI

### Installation Steps
1. Check the plugins to install
2. Click "Download now and install after restart"
3. Restart Jenkins when installation completes

---

## Tool Configuration

### 1. Configure JDK

Navigate to: `Manage Jenkins` > `Global Tool Configuration` > `JDK`

1. Click "Add JDK"
2. Name: `JDK 11`
3. Uncheck "Install automatically" if you have JDK installed
4. JAVA_HOME: `C:\Program Files\Eclipse Adoptium\jdk-11.0.29.7-hotspot` (Windows)
   - Or `/usr/lib/jvm/java-11-openjdk` (Linux)
5. Click "Save"

### 2. Configure Gradle

Navigate to: `Manage Jenkins` > `Global Tool Configuration` > `Gradle`

1. Click "Add Gradle"
2. Name: `Gradle 8.4`
3. Check "Install automatically"
4. Version: `Gradle 8.4`
5. Click "Save"

### 3. Configure Git

Navigate to: `Manage Jenkins` > `Global Tool Configuration` > `Git`

1. Name: `Default`
2. Path to Git executable: `git` (or full path if not in PATH)
3. Click "Save"

---

## Credentials Setup

Navigate to: `Manage Jenkins` > `Manage Credentials` > `(global)` > `Add Credentials`

### 1. SonarQube Token
- **Kind**: Secret text
- **Scope**: Global
- **Secret**: [Your SonarQube token]
- **ID**: `sonar-token`
- **Description**: SonarQube authentication token

### 2. MyMavenRepo Credentials
- **Kind**: Username with password
- **Scope**: Global
- **Username**: `myMavenRepo`
- **Password**: `test0005`
- **ID**: `mymavenrepo-credentials`
- **Description**: MyMavenRepo deployment credentials

### 3. GitHub Credentials (if repository is private)
- **Kind**: Username with password
- **Scope**: Global
- **Username**: [Your GitHub username]
- **Password**: [Your GitHub personal access token]
- **ID**: `github-credentials`
- **Description**: GitHub repository access

---

## SonarQube Configuration

### 1. Install SonarQube

#### Using Docker (Recommended)
```bash
docker run -d --name sonarqube -p 9000:9000 sonarqube:latest
```

#### Manual Installation
1. Download from https://www.sonarqube.org/downloads/
2. Extract and run:
   ```bash
   # Windows
   bin\windows-x86-64\StartSonar.bat
   
   # Linux/Mac
   bin/linux-x86-64/sonar.sh start
   ```

### 2. Configure SonarQube

1. Open http://localhost:9000
2. Login with default credentials: `admin` / `admin`
3. Change password when prompted
4. Create a new project:
   - Project key: `matrix-api`
   - Display name: `Matrix API`
5. Generate token:
   - My Account > Security > Generate Tokens
   - Name: `jenkins`
   - Click "Generate"
   - **Copy the token** (you'll need it for Jenkins)

### 3. Configure SonarQube in Jenkins

Navigate to: `Manage Jenkins` > `Configure System` > `SonarQube servers`

1. Check "Enable injection of SonarQube server configuration"
2. Click "Add SonarQube"
3. Name: `SonarQube`
4. Server URL: `http://localhost:9000`
5. Server authentication token: Select `sonar-token` credential
6. Click "Save"

---

## Project Setup

### 1. Create Multibranch Pipeline

1. From Jenkins dashboard, click **New Item**
2. Enter name: `Matrix-API-Pipeline`
3. Select: **Multibranch Pipeline**
4. Click **OK**

### 2. Configure Branch Sources

**Branch Sources Section:**
1. Click **Add source** > **Git**
2. Project Repository: `https://github.com/your-username/your-repo.git`
3. Credentials: Select GitHub credentials (if private)

**Behaviors:**
- Click **Add** > **Discover branches**
- Click **Add** > **Discover pull requests from origin**
  - Strategy: "The current pull request revision"
- Click **Add** > **Clean before checkout**

### 3. Build Configuration

**Build Configuration Section:**
- Mode: `by Jenkinsfile`
- Script Path: `Jenkinsfile`

### 4. Scan Multibranch Pipeline Triggers

Check: **Periodically if not otherwise run**
- Interval: `1 hour` (or as needed)

### 5. Save Configuration

Click **Save**

---

## GitHub Webhook Configuration

### 1. Get Jenkins Webhook URL

Format: `http://YOUR_JENKINS_URL/github-webhook/`
Example: `http://jenkins.example.com:8080/github-webhook/`

**Note**: If Jenkins is on localhost, you'll need to expose it:
- Use ngrok: `ngrok http 8080`
- Or use a cloud Jenkins instance

### 2. Configure Webhook in GitHub

1. Go to your GitHub repository
2. Click **Settings** > **Webhooks** > **Add webhook**
3. Configure:
   - **Payload URL**: `http://YOUR_JENKINS_URL/github-webhook/`
   - **Content type**: `application/json`
   - **SSL verification**: Enable (if using HTTPS)
   - **Which events**: Select "Just the push event"
   - **Active**: Check this option
4. Click **Add webhook**

### 3. Test Webhook

1. Make a commit and push to your repository
2. Check GitHub webhook delivery (Settings > Webhooks > Recent Deliveries)
3. Jenkins should automatically trigger a build

---

## Email Notification Configuration

### 1. Configure SMTP in Jenkins

Navigate to: `Manage Jenkins` > `Configure System` > `Extended E-mail Notification`

**For Gmail:**
- SMTP server: `smtp.gmail.com`
- SMTP Port: `587`
- Credentials: Add Gmail credentials with App Password
- Use SSL: Yes
- Use TLS: Yes
- Charset: `UTF-8`

### 2. Test Email

Click "Test configuration by sending test e-mail"

---

## Slack Notification Configuration

### 1. Create Slack App

1. Go to https://api.slack.com/apps
2. Click "Create New App"
3. Choose "From scratch"
4. Name: `Jenkins CI/CD`
5. Select your workspace

### 2. Enable Incoming Webhooks

1. Click "Incoming Webhooks"
2. Toggle "Activate Incoming Webhooks" to On
3. Click "Add New Webhook to Workspace"
4. Select channel (e.g., `#jenkins-notifications`)
5. Click "Allow"
6. **Copy the Webhook URL**

### 3. Configure Slack in Jenkins

Navigate to: `Manage Jenkins` > `Configure System` > `Slack`

1. Workspace: `Your workspace name`
2. Credential: Add Slack token/webhook
3. Default channel: `#jenkins-notifications`
4. Test Connection

### 4. Update Jenkinsfile

Replace the Slack webhook URL in the Jenkinsfile:
```groovy
SLACK_CHANNEL = '#jenkins-notifications'
```

---

## Testing the Pipeline

### 1. Trigger First Build

Option 1: **Manual Trigger**
- Go to your pipeline in Jenkins
- Click "Scan Multibranch Pipeline Now"

Option 2: **Git Push**
```bash
git add .
git commit -m "test: trigger Jenkins pipeline"
git push origin main
```

### 2. Monitor Build

1. Click on the running build
2. Click "Console Output" to see logs
3. Monitor each stage:
   - ✅ Checkout
   - ✅ Test
   - ✅ Code Analysis
   - ✅ Code Quality Gate
   - ✅ Build
   - ✅ Deploy
   - ✅ Notification

### 3. View Reports

After successful build:
- **Cucumber Report**: Build > Cucumber Report
- **JUnit Report**: Build > Test Result
- **JaCoCo Coverage**: Build > Coverage Report
- **Javadoc**: Build > Javadoc
- **SonarQube**: Click link in console or visit SonarQube dashboard

---

## Troubleshooting

### Build Fails at Test Stage
```bash
# Run tests locally first
./gradlew clean test

# Check test reports
build/reports/tests/test/index.html
```

### SonarQube Connection Failed
- Verify SonarQube is running: http://localhost:9000
- Check token is correct
- Review Jenkins system logs

### Quality Gate Fails
- Review SonarQube dashboard
- Adjust Quality Gate thresholds
- Fix code issues and re-run

### Deployment Fails
- Verify MyMavenRepo credentials
- Check network connectivity
- Review repository permissions

### Webhook Not Triggering
- Check webhook delivery in GitHub
- Verify Jenkins URL is accessible
- Check Jenkins logs: `Manage Jenkins` > `System Log`

---

## Best Practices

1. **Security**
   - Never commit credentials to repository
   - Use Jenkins credentials store
   - Enable CSRF protection
   - Use HTTPS in production

2. **Pipeline**
   - Use declarative pipeline syntax
   - Add timeout limits to stages
   - Implement proper error handling
   - Archive important artifacts

3. **Testing**
   - Run tests locally before pushing
   - Maintain high code coverage (>80%)
   - Fix failing tests immediately

4. **Notifications**
   - Configure meaningful messages
   - Use different channels for different severity
   - Include relevant build information

---

## Additional Resources

- [Jenkins Documentation](https://www.jenkins.io/doc/)
- [Pipeline Syntax](https://www.jenkins.io/doc/book/pipeline/syntax/)
- [SonarQube Documentation](https://docs.sonarqube.org/)
- [Gradle Documentation](https://docs.gradle.org/)

---

## Support

For issues or questions:
- Check Jenkins logs: `Manage Jenkins` > `System Log`
- Review build console output
- Check plugin documentation
- Contact team lead or instructor

---

**Created for: TP N°7 – Intégration continue**  
**École nationale Supérieure d'Informatique - 2CS SIL**
