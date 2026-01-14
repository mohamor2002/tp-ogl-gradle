# Fix Jenkins Java Version Error

## Problem
```
java.lang.UnsupportedClassVersionError: class file version 65.0
this version of the Java Runtime only recognizes class file versions up to 61.0
```

This means Jenkins plugins need Java 21, but Jenkins is running on Java 17.

## Solution: Upgrade Jenkins to Java 21

### Step 1: Download Java 21

1. Download Java 21 from: https://adoptium.net/temurin/releases/?version=21
2. Choose Windows x64 MSI installer
3. Install to: `C:\Program Files\Eclipse Adoptium\jdk-21.x.x.x-hotspot`

### Step 2: Update Jenkins to Use Java 21

#### If Jenkins is running as Windows Service:

1. Stop Jenkins service:
   ```cmd
   net stop jenkins
   ```

2. Edit `C:\Program Files\Jenkins\jenkins.xml`

3. Find the `<executable>` tag and update it:
   ```xml
   <executable>C:\Program Files\Eclipse Adoptium\jdk-21.0.5.11-hotspot\bin\java.exe</executable>
   ```

4. Start Jenkins:
   ```cmd
   net start jenkins
   ```

#### If Jenkins is running as WAR file:

1. Stop Jenkins (Ctrl+C in the terminal)

2. Set JAVA_HOME:
   ```cmd
   set JAVA_HOME=C:\Program Files\Eclipse Adoptium\jdk-21.0.5.11-hotspot
   set PATH=%JAVA_HOME%\bin;%PATH%
   ```

3. Restart Jenkins:
   ```cmd
   java -jar jenkins.war --httpPort=8080
   ```

### Step 3: Verify Java Version

1. Open Jenkins: http://localhost:8080
2. Go to: Manage Jenkins > System Information
3. Check `java.version` - should be 21.x.x

### Step 4: Restart Jenkins Pipeline

1. Go to your pipeline project
2. Click "Scan Multibranch Pipeline Now"
3. The build should now start successfully

## Alternative: Use Jenkins Docker with Java 21

If you prefer using Docker:

```bash
docker run -d -p 8080:8080 -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  --name jenkins \
  jenkins/jenkins:lts-jdk21
```

## Verify the Fix

After upgrading, run:
```cmd
java -version
```

Should output:
```
openjdk version "21.0.x" 2024-xx-xx
OpenJDK Runtime Environment Temurin-21.0.x+x (build 21.0.x+x)
```

Then check Jenkins System Information for the same version.
