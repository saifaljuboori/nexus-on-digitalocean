# Nexus on DigitalOcean

## Description

This project demonstrates how to install, configure, and manage Nexus Repository Manager on an Ubuntu server hosted on DigitalOcean Droplet. The project covers configuring Nexus, creating a user and permission, and publishing Java sample applications built with Gradle and Maven to Nexus hosted repositories.

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
Operating System = Ubuntu Linux
CPU = 4 vCPUs
Memory = 8 GB RAM
Nexus Port = 8081

A minimum configuration of 2 vCPUs and 4 GB RAM is recommended for this demo environment to avoid performance issues during Nexus installation and operation.

### Server Preparation

After creating the Droplet and configuring SSH access, the server environment was prepared.

#### Install OpenJDK 17

OpenJDK 17 was installed to provide a consistent Java environment for Nexus and the sample Java applications (Gradle and Maven) included in this repository.

Verify installation:

```
java -version
```

Expected Outcome:

`openjdk version "17.0.19"`

