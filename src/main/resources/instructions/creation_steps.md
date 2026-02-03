# Spring Cloud Config Server - Setup Guide

## Overview
This is a Spring Cloud Config Server that supports both **local file system** and **GitHub** as configuration sources. You can switch between them easily using a property.

---

## 1. Project Setup ✅

### Dependencies (pom.xml)
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>

<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>2023.0.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

### Main Application Class
```java
package com.routine.config;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.config.server.EnableConfigServer;

@SpringBootApplication
@EnableConfigServer  // This enables Config Server functionality
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

---

## 2. Configuration Files

### application.properties
```properties
spring.application.name=config-server
server.port=8888

# CONTROL VARIABLE: Set to 'local' or 'github'
# config.source=local
config.source=github

# Enable the profile based on source
spring.profiles.active=${config.source}

# Define config paths for each microservice
config.path.routine.sport=routine/sport
config.path.routine.culture=routine/culture
```

### application-local.properties (Local File System)
```properties
# Use native profile for local file system
spring.profiles.include=native

# Search locations for config files
# Option 1: Absolute path on your system
spring.cloud.config.server.native.search-locations=file:///C:/Users/hp/Desktop/WORKSPACE/config-repo

# Option 2: Classpath (if storing config files in resources folder)
# spring.cloud.config.server.native.search-locations=classpath:/config-repo

# Optional: Add multiple search locations
# spring.cloud.config.server.native.search-locations=file:///path1,file:///path2
```

### application-github.properties (GitHub Repository)
```properties
# Git repository URL
spring.cloud.config.server.git.uri=https://github.com/your-username/config-repo

# Default branch to use
spring.cloud.config.server.git.default-label=main

# For private repositories, add credentials:
# spring.cloud.config.server.git.username=your-github-username
# spring.cloud.config.server.git.password=your-personal-access-token

# Optional: Clone repository to specific directory
# spring.cloud.config.server.git.clone-on-start=true
# spring.cloud.config.server.git.basedir=target/config-repo

# Optional: Skip SSL verification (not recommended for production)
# spring.cloud.config.server.git.skip-ssl-validation=true
```

---

## 3. Configuration Repository Structure

Create a configuration repository with the following structure:

```
config-repo/
├── application.properties              # Default config for ALL services
├── application-dev.properties          # Dev environment for all services
├── application-prod.properties         # Prod environment for all services
│
├── sport-service.properties            # Sport service default config
├── sport-service-dev.properties        # Sport service dev config
├── sport-service-prod.properties       # Sport service prod config
│
├── culture-service.properties          # Culture service default config
├── culture-service-dev.properties      # Culture service dev config
└── culture-service-prod.properties     # Culture service prod config
```

### Example Config Files

**application.properties** (Shared by all services)
```properties
# Common properties
logging.level.root=INFO
spring.jackson.serialization.indent-output=true
```

**sport-service.properties**
```properties
service.name=Sport Service
service.description=Handles all sport-related operations
sport.max-players=11
sport.timeout=30000
```

**sport-service-dev.properties**
```properties
# Development environment settings
spring.datasource.url=jdbc:mysql://localhost:3306/sport_dev
spring.datasource.username=dev_user
spring.datasource.password=dev_password
debug=true
```

**sport-service-prod.properties**
```properties
# Production environment settings
spring.datasource.url=jdbc:mysql://prod-server:3306/sport_prod
spring.datasource.username=prod_user
spring.datasource.password=${DB_PASSWORD}
debug=false
```

---

## 4. Setup Instructions

### For Local File System

1. **Create the config repository folder:**
   ```bash
   mkdir C:\Users\hp\Desktop\WORKSPACE\config-repo
   ```

2. **Add your configuration files** to this folder

3. **Update application.properties:**
   ```properties
   config.source=local
   ```

4. **Update application-local.properties** with correct path

5. **Start the server:**
   ```bash
   mvn spring-boot:run
   ```

### For GitHub

1. **Create a GitHub repository** named `config-repo`

2. **Push your configuration files** to the repository:
   ```bash
   cd config-repo
   git init
   git add .
   git commit -m "Initial config"
   git branch -M main
   git remote add origin https://github.com/your-username/config-repo.git
   git push -u origin main
   ```

3. **Update application.properties:**
   ```properties
   config.source=github
   ```

4. **Update application-github.properties** with your GitHub details

5. **Start the server:**
   ```bash
   mvn spring-boot:run
   ```

---

## 5. Testing the Config Server

Once the server is running on port 8888, test it using these URLs:

### Access Patterns
```
http://localhost:8888/{application}/{profile}
http://localhost:8888/{application}/{profile}/{label}
```

### Example URLs

**Get sport-service default config:**
```
http://localhost:8888/sport-service/default
```

**Get sport-service dev config:**
```
http://localhost:8888/sport-service/dev
```

**Get sport-service prod config:**
```
http://localhost:8888/sport-service/prod
```

**Get culture-service config:**
```
http://localhost:8888/culture-service/dev
```

**Get specific branch/tag (GitHub only):**
```
http://localhost:8888/sport-service/dev/feature-branch
```

### Response Format
The response will be in JSON format:
```json
{
  "name": "sport-service",
  "profiles": ["dev"],
  "label": null,
  "version": null,
  "state": null,
  "propertySources": [
    {
      "name": "file:///C:/Users/hp/Desktop/WORKSPACE/config-repo/sport-service-dev.properties",
      "source": {
        "spring.datasource.url": "jdbc:mysql://localhost:3306/sport_dev",
        "debug": "true"
      }
    }
  ]
}
```

---

## 6. Client Service Configuration

To use this config server in your microservices, add these dependencies and configurations:

### Client Dependencies (pom.xml)
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

### Client Configuration (application.properties or bootstrap.properties)
```properties
spring.application.name=sport-service
spring.profiles.active=dev

# Config Server URL
spring.config.import=optional:configserver:http://localhost:8888

# Alternative (older approach using bootstrap.properties):
# spring.cloud.config.uri=http://localhost:8888
# spring.cloud.config.name=sport-service
# spring.cloud.config.profile=dev

# Enable refresh endpoint
management.endpoints.web.exposure.include=refresh,health,info
```

---

## 7. Switching Between Local and GitHub

To switch the configuration source:

1. **Open** `application.properties`
2. **Change** the `config.source` property:
   - For local: `config.source=local`
   - For GitHub: `config.source=github`
3. **Restart** the config server

---

## 8. Advanced Features

### Refresh Configuration Without Restart

Clients can refresh their configuration using the `/actuator/refresh` endpoint:
```bash
curl -X POST http://localhost:8080/actuator/refresh
```

### Encrypt/Decrypt Properties

Add encryption key in application.properties:
```properties
encrypt.key=my-secret-key
```

Encrypt a value:
```bash
curl http://localhost:8888/encrypt -d mysecret
```

Use encrypted value in config files:
```properties
password={cipher}AQA3eYXHqW...
```

### Health Check

Check config server health:
```
http://localhost:8888/actuator/health
```

---

## 9. Troubleshooting

### Issue: Config not loading
- Check server logs for errors
- Verify config file naming: `{application-name}-{profile}.properties`
- Ensure config server is running on port 8888
- Test URLs directly in browser

### Issue: GitHub authentication failed
- Use Personal Access Token instead of password
- Verify repository URL and branch name
- Check firewall/proxy settings

### Issue: File not found (local)
- Verify absolute path in `application-local.properties`
- Use forward slashes: `file:///C:/path/to/config`
- Check file permissions

---

## 10. Production Considerations

1. **Security:**
   - Use Spring Security to protect config server
   - Encrypt sensitive properties
   - Use HTTPS in production

2. **High Availability:**
   - Run multiple instances behind a load balancer
   - Use Spring Cloud Bus for distributed refresh

3. **Monitoring:**
   - Enable Spring Boot Actuator endpoints
   - Monitor config server health and metrics

4. **Git Strategy:**
   - Use protected branches
   - Implement change approval process
   - Tag stable configurations

---

## Summary

✅ Config Server running on port **8888**  
✅ Supports **Local** and **GitHub** sources  
✅ Easy switching with `config.source` property  
✅ Multiple microservices and profiles supported  
✅ RESTful API for accessing configurations  

**Current Status:** Ready to use! Switch between local and GitHub by changing `config.source` in application.properties.
