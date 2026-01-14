# Matrix API - Jenkins CI/CD Pipeline

## Project Description
This is a Java-based Matrix API project with automated continuous integration and deployment using Jenkins.

## Project Structure
```
.
├── src/
│   ├── main/java/com/example/
│   │   ├── model/Matrix.java
│   │   ├── service/MatrixMathematics.java
│   │   └── exception/NoSquareException.java
│   └── test/java/
│       ├── MatrixSteps.java
│       └── ExampleTest.java
├── Features/
│   └── matrix.feature
├── build.gradle
├── Jenkinsfile
└── README.md
```

## Prerequisites

### 1. Jenkins Setup
- Jenkins 2.x or higher
- Required Jenkins Plugins:
  - Pipeline
  - Git
  - Gradle Plugin
  - SonarQube Scanner
  - JaCoCo Plugin
  - HTML Publisher Plugin
  - JUnit Plugin
  - Email Extension Plugin
  - Slack Notification Plugin

### 2. Tools Configuration in Jenkins
- **JDK 11**: Configure in `Manage Jenkins` > `Global Tool Configuration`
- **Gradle 8.4**: Configure in `Manage Jenkins` > `Global Tool Configuration`
- **SonarQube**: Configure in `Manage Jenkins` > `Configure System`

### 3. SonarQube Setup
- Install and run SonarQube server (default: http://localhost:9000)
- Create a project token in SonarQube
- Add the token to Jenkins credentials as `sonar-token`

### 4. Credentials Setup in Jenkins
Add the following credentials in `Manage Jenkins` > `Credentials`:

1. **SonarQube Token** (Secret text)
   - ID: `sonar-token`
   - Description: SonarQube authentication token

2. **MyMavenRepo Credentials** (Username with password)
   - ID: `mymavenrepo-credentials`
   - Username: `myMavenRepo`
   - Password: `test0005`

## Jenkins Pipeline Configuration

### 1. Create Multibranch Pipeline

1. In Jenkins, click **New Item**
2. Enter a name (e.g., "Matrix-API-Pipeline")
3. Select **Multibranch Pipeline**
4. Click **OK**

### 2. Configure Branch Sources

1. Under **Branch Sources**, click **Add source** > **Git**
2. Enter your GitHub repository URL
3. Add credentials if repository is private
4. Under **Behaviors**, add:
   - Discover branches
   - Discover pull requests from origin
   - Clean before checkout

### 3. Build Configuration

- **Script Path**: `Jenkinsfile` (default)
- **Scan Multibranch Pipeline Triggers**: Check "Periodically if not otherwise run" (e.g., 1 hour)

### 4. GitHub Webhook Configuration

1. Go to your GitHub repository
2. Navigate to **Settings** > **Webhooks** > **Add webhook**
3. Set Payload URL: `http://<JENKINS_URL>/github-webhook/`
4. Content type: `application/json`
5. Select: "Just the push event"
6. Check "Active"
7. Click **Add webhook**

## Pipeline Stages

The Jenkins pipeline consists of the following stages:

### 1. **Test Phase**
- Runs unit tests using JUnit
- Executes Cucumber BDD tests
- Generates and archives test reports
- Publishes HTML reports for Cucumber tests

### 2. **Code Analysis Phase**
- Performs static code analysis using SonarQube
- Generates code quality metrics
- Analyzes code coverage with JaCoCo

### 3. **Code Quality Gate Phase**
- Waits for SonarQube Quality Gate result
- **Stops pipeline if Quality Gate fails**
- Ensures code meets quality standards

### 4. **Build Phase**
- Compiles the Java code
- Generates JAR artifact
- Creates Javadoc documentation
- Archives artifacts
- Publishes JaCoCo coverage reports

### 5. **Deploy Phase**
- Publishes JAR to MyMavenRepo
- Includes main JAR, sources, and Javadoc

### 6. **Notification Phase**
- **On Success**: Sends notifications via Email and Slack
- **On Failure**: Sends error notifications to development team
- Uses both custom Gradle tasks and Jenkins plugins

## Local Build Commands

### Run Tests
```bash
./gradlew test
```

### Generate Cucumber Reports
```bash
./gradlew generateCucumberReports
```

### Run SonarQube Analysis
```bash
./gradlew sonar
```

### Build JAR
```bash
./gradlew jar
```

### Generate Javadoc
```bash
./gradlew javadoc
```

### Publish to Maven Repository
```bash
./gradlew publish
```

### Clean Build
```bash
./gradlew clean build
```

## Notification Configuration

### Email Notifications
Edit `build.gradle` to configure email settings:
```gradle
def emailFrom = 'your-email@example.com'
def emailPassword = 'your-app-password'
def emailTo = 'recipient@example.com'
```

### Slack Notifications
Edit `build.gradle` to configure Slack webhook:
```gradle
def slackWebhookUrl = 'https://hooks.slack.com/services/YOUR/WEBHOOK/URL'
```

Also update the Jenkinsfile environment variable:
```groovy
SLACK_CHANNEL = '#your-channel'
```

## Quality Gates

The pipeline enforces the following quality standards:
- Code coverage thresholds
- Code smell limits
- Security vulnerability checks
- Duplication limits
- Maintainability ratings

Configure these in SonarQube under **Quality Gates**.

## Troubleshooting

### Pipeline Fails at Code Quality Gate
- Check SonarQube server is running
- Verify Quality Gate conditions in SonarQube
- Review code analysis results

### Deployment Fails
- Verify MyMavenRepo credentials
- Check repository URL is accessible
- Ensure network connectivity

### Tests Fail
- Run tests locally: `./gradlew test`
- Check test reports in `build/reports/tests/test/`
- Review Cucumber reports in `build/reports/cucumber/`

### Notifications Not Sent
- Verify email SMTP settings
- Check Slack webhook URL is valid
- Review Jenkins console output for error messages

## Viewing Reports

After a successful build, the following reports are available in Jenkins:

1. **JUnit Test Report**: Shows unit test results
2. **Cucumber Report**: BDD test scenarios and results
3. **JaCoCo Coverage Report**: Code coverage metrics
4. **Javadoc**: API documentation
5. **SonarQube Dashboard**: Comprehensive code quality analysis

## License
This project is for educational purposes (ESI - 2CS SIL).

## Authors
- TP N°7 – Intégration continue
- École nationale Supérieure d'Informatique
