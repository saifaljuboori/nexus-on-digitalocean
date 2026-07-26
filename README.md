# Nexus on DigitalOcean

## Description

This project demonstrates how to install, configure, and manage Nexus Repository Manager on an Ubuntu server hosted on DigitalOcean Droplet. The project covers configuring Nexus, creating a Nexus user and permissions, and publishing Java sample applications built with Gradle and Maven to Nexus hosted repositories.

## Technologies Used
  - DigitalOcean Droplet
  - Linux Ubuntu Server
  - Nexus Repository Manager
  - Java
  - Gradle
  - Maven
  - Git & GitHub
  - Bash

## Infrastructure Setup

### DigitalOcean Droplet Configuration
A DigitalOcean Droplet was provisioned to host Nexus Repository Manager. The Droplet settings were as following:
  - Operating System = Ubuntu Linux
  - CPU = 4 vCPUs
  - Memory = 8 GB RAM
  - Nexus Port = 8081

A minimum configuration of 2 vCPUs and 4 GB RAM is recommended for this demo environment to avoid performance issues during Nexus installation and operation.

### Server Preparation

After creating the Droplet and configuring SSH access, the server environment was prepared for Nexus installation.

#### Install OpenJDK 17

OpenJDK 17 was installed to provide a consistent Java environment for Nexus compatibility with the sample Java applications (Gradle and Maven) included in this repository.

```
sudo apt update
sudo apt install openjdk-17-jdk -y
```

Verify installation:

```
java -version
```

Expected Outcome:

`openjdk version "17.0.19"`

## Install and Configure Nexus Repository Manager

### Create Nexus User
Nexus should not run as the root user. A dedicated service account was created to run Nexus with limited permissions.

```
sudo adduser nexus
```
Switch to the Nexus user:
```
sudo su - nexus
```
### Download Nexus Repository Manager
Navigate to the installation directory:
```
cd /opt
```
Download Nexus Repository Manager:
```
wget https://download.sonatype.com/nexus/3/nexus-3.94.1-06-linux-x86_64.tar.gz
```
Extract the Nexus archive:
```
tar -xvzf nexus-3.94.1-06-linux-x86_64.tar.gz
```
The following directories are generated:
nexus >> contains Nexus application runtime files
sonatype-work >> contains Nexus repository data and configuration

### Assign Ownership to Nexus User

The Nexus user must own both the application directory and the data directory.

Change ownership:

```
sudo chown -R nexus:nexus /opt/nexus
sudo chown -R nexus:nexus /opt/sonatype-work
```
Verify ownership:
```
ls -l /opt
```
Expected result:
```
drwxr-xr-x nexus nexus nexus
drwxr-xr-x nexus nexus sonatype-work
```
### Configure Nexus to Run as Nexus User

Edit the Nexus configuration file:

```
vim /opt/nexus/bin/nexus.rc
```

Set the runtime user:
```
run_as_user="nexus"
```
This ensures Nexus starts using the dedicated service account.

### Start Nexus Repository Manager

Start Nexus:

```
/opt/nexus/bin/nexus start
```

Check Nexus status:
```
/opt/nexus/bin/nexus status
```

Verify that Nexus is listening on port 8081:
```
netstat -ltnp | grep 8081
```

Expected output:
```
tcp6  0  0 :::8081  :::*  LISTEN  java
```

This ensures Nexus starts using the dedicated service account.

### Configure Droplet Firewall
On the DigitalOcean Firewall, configure the inbound rules to allow incoming traffic on port 8081 so nexus can be accessed externally.

### Access Nexus Repository Manager

Nexus can be accessed from a browser using:

```
http://<server-ip>:8081
```

The initial administrator password can be retrieved from:

```
cat /opt/sonatype-work/nexus3/admin.password
```
Use this password for the first login and complete the Nexus setup wizard.

## Nexus Security Configuration

### Create Nexus Privileges

Nexus Repository Manager uses a role-based access control (RBAC) model where privileges are assigned to roles, and roles are assigned to users.

Custom privileges were created to provide the required access for publishing and managing Maven artifacts.

The following privileges were configured:

| Privilege | Purpose |
|---|---|
| `nx-repository-view-maven2-*-*` | Allows viewing and accessing Maven repositories |
| `nx-repository-admin-maven2-maven-central-*` | Allows administrative operations on Maven repositories |

The privileges were created based on the required operations for publishing Java artifacts built with Maven and Gradle.

### Create Nexus Role

A custom role was created to group the required repository privileges.

Navigation:
```
Administration → Security → Roles → Create Role
```

Role configuration:

| Setting | Value |
|---|---|
| Role ID | `nx` |
| Role Name | `nx-java` |
| Assigned Privileges | `nx-repository-view-maven2-*-*` |
|  | `nx-repository-admin-maven2-maven-central-*` |

Using roles instead of assigning privileges directly to users provides centralized permission management and makes future access changes easier to maintain.

### Create Nexus User

A dedicated Nexus user was created for artifact publishing.

Navigation:

```
Administration → Security → Users → Create User
```


User configuration:

| Setting | Value |
|---|---|
| Username | `saif` |
| Assigned Role | `nx-java` |

The user receives repository access through the assigned role rather than through direct privilege assignment.

## Publish Java Gradle Application

The Gradle sample application included in this repository demonstrates how to build a Java application and publish the generated JAR artifact to a Nexus Maven hosted repository.

The publishing workflow follows this process:
Java Source Code
|
v
Gradle Build
|
v
Generate JAR Artifact
|
v
Gradle Publish
|
v
Nexus Maven Hosted Repository

### Configure Gradle Maven Publishing

The Gradle project uses the `maven-publish` plugin to publish the generated Java artifact to Nexus Repository Manager.

The publishing configuration is defined in:

```
java-gradle-app/build.gradle
```


The project defines a Maven publication that publishes the generated JAR file:

```
publishing {
    publications {
        create("maven", MavenPublication) {
            artifact("build/libs/my-app-$version.jar") {
                extension 'jar'
            }
        }
    }
}
```
The artifact version is generated from the Gradle project version configuration, and the resulting JAR file is uploaded to the Nexus Maven hosted repository.

Configure Nexus Repository

The Nexus repository configuration is defined in the Gradle publishing repository block:

repositories {
    maven {
        name 'nexus'
        url "http://<nexus-server-ip>:8081/repository/maven-repo/"
        allowInsecureProtocol = true

        credentials {
            username project.repoUser
            password project.repoPassword
        }
    }
}

Repository configuration in Nexus UI:

| Setting         | Value        |
| --------------- | ------------ |
| Repository Name | `maven-repo` |
| Repository Type | Maven Hosted |
| Protocol        | HTTP         |
| Port            | `8081`       |

The `allowInsecureProtocol` setting is required because the Nexus instance is configured using HTTP. Gradle blocks insecure HTTP repository communication unless it is explicitly allowed.

Configure Nexus Credentials

Authentication credentials are stored outside the project source code.

The Gradle project reads credentials from:

```
gradle.properties
```
Example:

repoUser=<nexus-user>
repoPassword=<nexus-password>

These values are referenced inside `build.gradle`:

credentials {
    username project.repoUser
    password project.repoPassword
}

The credentials file should never be committed to GitHub.

Build the Java Application

The application is built using Gradle:

```
gradle build
```

The generated artifact is created under:
```
build/libs/
```

Example:
```
build/libs/my-app-1.0.0.jar
```
Publish Artifact to Nexus

After successfully building the application, the JAR artifact is published to Nexus:
```
gradle publish
```

The artifact is published to:
```
http://<nexus-server-ip>:8081/repository/maven-repo/
```
Verify Artifact in Nexus Repository Manager

After publishing:

  - Login to Nexus Repository Manager
  - Navigate to:
```
Browse → maven-repo
```
  - Verify that the Java application artifact is available.
The artifact is now stored in Nexus and can be consumed by other applications through the Maven repository URL.

## Publish Java Maven Application

The Maven sample application included in this repository demonstrates how to build a Java application and deploy the generated JAR artifact to a Nexus Maven snapshot repository.

The deployment workflow follows this process:


Java Source Code
|
v
Maven Build
|
v
Generate JAR Artifact
|
v
Maven Deploy
|
v
Nexus Maven Snapshot Repository


### Configure Maven Nexus Repository

The Maven project uses Maven's deployment mechanism to upload the generated Java artifact to Nexus Repository Manager.

The repository configuration is defined inside:
```
java-maven-app/pom.xml
```

The project is configured to deploy snapshot artifacts using the `distributionManagement` section:

```
<distributionManagement>
    <snapshotRepository>
        <id>nexus-snapshots</id>
        <url>http://<nexus-server-ip>:8081/repository/maven-snapshots/</url>
    </snapshotRepository>
</distributionManagement>
```
Repository configuration in Nexus UI:

| Setting         | Value             |
| --------------- | ----------------- |
| Repository Name | `maven-snapshots` |
| Repository Type | Maven Hosted      |
| Deployment Type | Snapshot          |
| Protocol        | HTTP              |
| Port            | `8081`            |

The repository is configured as a snapshot repository because the Maven project version contains the `SNAPSHOT` identifier.

Maven uses the `snapshotRepository` configuration when deploying snapshot versions.

Create Maven Settings Configuration

Maven requires authentication credentials to upload artifacts to Nexus.

Credentials are stored in Maven's user-level configuration file:
```
~/.m2/settings.xml
```
Create the Maven configuration directory:
```
mkdir -p ~/.m2
```
Create the Maven settings file:
```
vim ~/.m2/settings.xml
```
Add the Nexus authentication configuration:

<settings>
    <servers>
        <server>
            <id>nexus-snapshots</id>
            <username><nexus-user></username>
            <password><nexus-password></password>
        </server>
    </servers>
</settings>

The <id> value must match the repository ID configured in `pom.xml`:
<id>nexus-snapshots</id>

Maven uses this matching ID to locate the correct credentials when deploying artifacts.

The `settings.xml` file is stored outside the project repository and should never be committed to GitHub because it contains sensitive authentication information.

Build the Maven Application

The Maven application was built using:
```
mvn package
```

The generated artifact is available under:
```
target/my-app-1.0-SNAPSHOT.jar
```

Deploy Artifact to Nexus

After successfully building the application, the JAR artifact is deployed to the Nexus Maven snapshot repository.

Execute:
```
mvn deploy
```

The artifact is deployed to:
```
http://<nexus-server-ip>:8081/repository/maven-snapshots/
```
Verify Artifact in Nexus Repository Manager

After deploying the artifact:

  - Login to Nexus Repository Manager
  - Navigate to:
```
Browse → maven-snapshots
```
  - Locate the uploaded Maven artifact.
The Java Maven artifact is now stored in Nexus and can be consumed by other applications through the configured Maven repository URL.
