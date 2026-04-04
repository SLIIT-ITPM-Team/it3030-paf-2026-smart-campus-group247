# 🚀 How to Run Smart Campus Operations Hub Backend

## Prerequisites Checklist

Before running the project, ensure you have:

- [ ] **Java 25 (LTS)** - Check: `java -version`
- [ ] **Maven 3.6+** - Check: `mvn -version`
- [ ] **MySQL 8.0+** - Running and accessible at localhost:3306
- [ ] **Git** (optional) - For version control

---

## Step 1: Install Prerequisites (Windows)

### 1.1 Install Java 25 (LTS)

**Option A: Using Chocolatey (if installed)**
```powershell
choco install openjdk25
```

If you get `choco` not recognized, use Option B.

**Option B: Using winget (recommended on Windows 10/11)**
```powershell
winget install EclipseAdoptium.Temurin.25.JDK
```

**Option C: Manual Installation**
1. Download from: https://www.oracle.com/java/technologies/downloads/#java25
2. Or use OpenJDK: https://adoptium.net/
3. Follow installation wizard
4. Verify: `java -version`

### 1.2 Install Maven

**Option A: Using Chocolatey (if installed)**
```powershell
choco install maven
```

If you get `choco` not recognized, use Option B.

**Option B: Using winget (recommended on Windows 10/11)**
```powershell
winget install Apache.Maven
```

**Option C: Manual Installation**
1. Download from: https://maven.apache.org/download.cgi
2. Extract to a location (e.g., `C:\Maven\apache-maven-3.9.0`)
3. Add to system PATH:
   - Open Environment Variables: Search "Environment Variables" in Windows
   - Edit System variables → New
   - Variable name: `MAVEN_HOME`
   - Variable value: `C:\Maven\apache-maven-3.9.0` (adjust path)
   - Then add `%MAVEN_HOME%\bin` to PATH
4. Verify: Open new terminal and run `mvn -version`

### 1.3 Verify MySQL

```powershell
# Check if MySQL is running
mysql -u root -p

# Or check status (Windows Service)
Get-Service | Where-Object {$_.Name -like "*MySQL*"}
```

---

## Step 2: Setup Database

```sql
-- Create the database
CREATE DATABASE IF NOT EXISTS smart_campus_db 
CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- Verify it was created
SHOW DATABASES;
```

**Note:** Hibernate will automatically create tables based on entity models during first run with `ddl-auto: update`.

---

## Step 3: Configure Database Credentials

**File:** `backend/smart-campus-api/src/main/resources/application.yml`

Update the datasource section:
```yaml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/smart_campus_db?useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=UTC
    username: root          # Your MySQL username
    password: your_password # Your MySQL password
```

Replace `your_password` with your actual MySQL password.

---

## Step 4: Build the Project

Open PowerShell or Command Prompt and navigate to the backend directory:

```powershell
# Navigate to project directory
cd "D:\PAF Project\Smart-Campus-Operations-Hub\backend\smart-campus-api"

# Clean and compile (skip tests)
mvn clean compile -DskipTests

# Or full build with dependencies
mvn clean install -DskipTests
```

**Expected Output:**
```
[INFO] BUILD SUCCESS
```

If you get "BUILD FAILURE", see [Troubleshooting](#troubleshooting) below.

---

## Step 5: Run the Application

### Method 1: Using Maven Spring Boot Plugin (Recommended)

```powershell
cd "D:\PAF Project\Smart-Campus-Operations-Hub\backend\smart-campus-api"
mvn spring-boot:run
```

**Expected Output - Final Lines:**
```
[INFO] Started SmartCampusApplication in X.XXX seconds (process running for X.XXX)
```

### Method 2: Run JAR File After Build

```powershell
# First build the project
mvn clean package -DskipTests

# Then run the JAR
java -jar target/smart-campus-api-0.0.1-SNAPSHOT.jar
```

### Method 3: Run from IDE (IntelliJ IDEA / VS Code)

**IntelliJ IDEA:**
1. Open project in IntelliJ
2. Right-click on `SmartCampusApplication.java`
3. Click "Run 'SmartCampusApplication.main()'"
4. Or press `Shift + F10`

**VS Code:**
1. Install "Extension Pack for Java" (Microsoft)
2. Open `SmartCampusApplication.java`
3. Click "Run" link above main method
4. Or press `Ctrl + F5`

---

## Step 6: Verify It's Running

The application should start on **http://localhost:8080**

### Test Health Endpoint

```powershell
# Using PowerShell
Invoke-WebRequest -Uri "http://localhost:8080/api/public/health" -UseBasicParsing | ConvertFrom-Json | ConvertTo-Json

# Or using curl (Git Bash or Windows 10+)
curl http://localhost:8080/api/public/health
```

**Expected Response:**
```json
{
  "status": "UP",
  "message": "Smart Campus Operations Hub API is running",
  "version": "1.0.0",
  "timestamp": "2026-04-02T...",
  "path": "/api/public/health"
}
```

### Test Authenticated Health Endpoint

```powershell
# This will return 401 Unauthorized (expected - requires authentication)
Invoke-WebRequest -Uri "http://localhost:8080/api/health" -UseBasicParsing
```

---

## Step 7: View Application Logs

Logs are configured in `application.yml`:

```yaml
logging:
  level:
    root: INFO
    com.smartcampus: DEBUG
    org.springframework.security: DEBUG
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} - %msg%n"
```

**To view logs while running:**
- All console output appears in the terminal where you ran `mvn spring-boot:run`
- Logs show SQL queries with `show-sql: true` and `format_sql: true`

---

## Troubleshooting

### ❌ Error: "mvn" is not recognized

**Solution:** Maven is not in PATH
- Reinstall Maven (see Step 1.2)
- Add Maven bin folder to system PATH
- Restart terminal/IDE
- Verify: `mvn -version`

### ❌ Error: "choco" is not recognized

**Solution:** Chocolatey is not installed or not on PATH
- Use winget instead:
  - `winget install EclipseAdoptium.Temurin.25.JDK`
  - `winget install Apache.Maven`
- Or install Chocolatey first (Admin PowerShell):
  - `Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))`
- Reopen terminal and verify:
  - `choco --version`

### ❌ Error: "Could not find or load main class"

**Solution:** Classes weren't compiled
- Run: `mvn clean compile -DskipTests`
- Check for Java compilation errors in output
- Verify Java 25 is installed: `java -version`

### ❌ Error: "Connection refused" (MySQL)

**Solution:** MySQL isn't running or credentials are wrong
- Verify MySQL is running: `mysql -u root -p -e "SELECT 1"`
- Check database exists: `SHOW DATABASES;`
- Update password in `application.yml`
- Verify URL: `jdbc:mysql://localhost:3306/smart_campus_db`

### ❌ Error: "java.nio.file.NoSuchFileException" (File uploads)

**Solution:** Upload directory doesn't exist
- Create directory: `D:\PAF Project\Smart-Campus-Operations-Hub\uploads`
- Or update `FileUploadUtil.UPLOAD_DIR` path in configuration

### ❌ Error: Port 8080 already in use

**Solution:** Another application is using port 8080
- Find what's using port 8080: `netstat -ano | findstr :8080`
- Either stop that application or change port in `application.yml`:
  ```yaml
  server:
    port: 8081
  ```

### ❌ Error: "Duplicate key error" in database

**Solution:** Hibernate schema conflicts
- Drop and recreate database:
  ```sql
  DROP DATABASE smart_campus_db;
  CREATE DATABASE smart_campus_db CHARACTER SET utf8mb4;
  ```
- Change `ddl-auto: create` temporarily in `application.yml`, run app, then revert to `update`

### ❌ Build takes too long / "Cannot resolve dependencies"

**Solution:** Maven is downloading dependencies
- Wait for download to complete (can take 5-10 minutes first time)
- Check internet connection
- Try: `mvn clean install -DskipTests -o` (offline mode after first run)
- Delete `~/.m2/repository` to clear cache and retry if downloads are corrupted

---

## Application Endpoints Reference

### Public Endpoints (No Authentication Required)
```
GET  http://localhost:8080/api/public/health        - Health check
GET  http://localhost:8080/api/auth/register        - Registration (Phase 2)
POST http://localhost:8080/api/auth/login           - Login (Phase 2)
```

### Protected Endpoints (Authentication Required)
```
GET  http://localhost:8080/api/health               - Authenticated health check
POST http://localhost:8080/api/auth/logout          - Logout (Phase 2)
POST http://localhost:8080/api/auth/refresh         - Refresh token (Phase 2)
```

### Development Tools
```
GET  http://localhost:8080/actuator                 - Spring Boot Actuator
GET  http://localhost:8080/actuator/health          - Detailed health
GET  http://localhost:8080/actuator/metrics         - Performance metrics
```

---

## Configuration Files

### Main Configuration
**File:** `src/main/resources/application.yml`
- Database connection
- JPA/Hibernate settings
- File upload limits
- OAuth2 settings
- Logging configuration

### POM Dependency Management
**File:** `pom.xml`
- Maven project configuration
- Spring Boot parent version (3.2.0)
- All dependencies with versions
- Build plugins

---

## Stop the Application

Press `Ctrl + C` in the terminal where the application is running.

**Expected Output:**
```
2026-04-02 14:30:00 - Shutting down...
```

---

## Environment Variables (Optional)

For production or custom configuration:

```powershell
# Set environment variables
$env:GOOGLE_CLIENT_ID = "your-google-client-id"
$env:GOOGLE_CLIENT_SECRET = "your-google-client-secret"
$env:JWT_SECRET = "your-jwt-secret-key"

# Then run application
mvn spring-boot:run
```

---

## Summary

| Step | Command | Time |
|------|---------|------|
| 1. Verify Java | `java -version` | < 1 min |
| 2. Install Maven | Download + PATH setup | 5 min |
| 3. Create database | `CREATE DATABASE` | < 1 min |
| 4. Build project | `mvn clean compile` | 2-5 min |
| 5. Run project | `mvn spring-boot:run` | < 1 min |
| 6. Verify health | `curl http://localhost:8080/api/public/health` | < 1 min |

**Total Time:** ~10-15 minutes (first time with downloads)

---

## Next Steps (Phase 2)

Once the application is running:

1. ✅ Verify all endpoints respond
2. ✅ Check MySQL database tables were created
3. ✅ Monitor logs for any errors
4. ✅ Begin Phase 2 development (entity models, services, REST APIs)

---

## Additional Resources

- **Spring Boot Docs:** https://spring.io/projects/spring-boot
- **Maven Docs:** https://maven.apache.org/
- **MySQL Docs:** https://dev.mysql.com/doc/
- **Java 25 Docs:** https://docs.oracle.com/en/java/javase/25/

---

## Support

If you encounter issues:
1. Check the [Troubleshooting](#troubleshooting) section
2. Review application logs (printed to console)
3. Verify all prerequisites are installed
4. Check the documentation files in `/docs/` folder

---

**Last Updated:** April 2, 2026
**Status:** Ready to Run ✅
