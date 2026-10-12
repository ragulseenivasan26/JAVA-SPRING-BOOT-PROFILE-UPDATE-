# Profile Update REST API (Spring Boot + Maven + Jenkins + Docker)

A beginner-friendly Spring Boot REST API that updates a profile and validates:
- `phoneNumber`: exactly 10 numeric digits.
- `address`: must not be empty or whitespace.

## Requirements
- Java 17+
- Maven 3.8+ (or use the Maven Wrapper if you add one)
- Git
- Optional: Docker
- Jenkins with Git and Maven/JDK configured

## Run locally
Open a terminal in the project root:

```bash
mvn clean test
mvn clean package
java -jar target/profile-update-api.jar
```

The API runs at `http://localhost:8080`.

## API endpoint

`PUT http://localhost:8080/api/profile`

Header: `Content-Type: application/json`

### Valid request
```json
{
  "phoneNumber": "9876543210",
  "address": "12 Lake Road, Chennai"
}
```

Expected status: `200 OK`

Example response:
```json
{
  "message": "Profile updated successfully",
  "phoneNumber": "9876543210",
  "address": "12 Lake Road, Chennai"
}
```

### Invalid phone number
```json
{
  "phoneNumber": "12345",
  "address": "12 Lake Road, Chennai"
}
```

Expected status: `400 Bad Request`, with a validation message under `errors.phoneNumber`.

### Empty address
```json
{
  "phoneNumber": "9876543210",
  "address": "   "
}
```

Expected status: `400 Bad Request`, with a validation message under `errors.address`.

## Test using curl

Valid input:
```bash
curl -i -X PUT http://localhost:8080/api/profile \
  -H "Content-Type: application/json" \
  -d "{\"phoneNumber\":\"9876543210\",\"address\":\"12 Lake Road, Chennai\"}"
```

Invalid phone:
```bash
curl -i -X PUT http://localhost:8080/api/profile \
  -H "Content-Type: application/json" \
  -d "{\"phoneNumber\":\"12345\",\"address\":\"12 Lake Road, Chennai\"}"
```

Empty address:
```bash
curl -i -X PUT http://localhost:8080/api/profile \
  -H "Content-Type: application/json" \
  -d "{\"phoneNumber\":\"9876543210\",\"address\":\"   \"}"
```

You can also use Postman: choose **PUT**, enter the URL above, then select **Body → raw → JSON**.

## Push source code to GitHub

1. Create a new empty repository on GitHub, for example `profile-update-api`.
2. Open a terminal in this project folder.
3. Run the following commands (replace the URL with your own repository URL):

```bash
git init
git add .
git commit -m "Initial Spring Boot profile update API"
git branch -M main
git remote add origin https://github.com/YOUR_USERNAME/profile-update-api.git
git push -u origin main
```

If Git says `remote origin already exists`, run `git remote set-url origin https://github.com/YOUR_USERNAME/profile-update-api.git` and retry the push.

## Jenkins Freestyle job setup

### 1. Configure tools
In Jenkins, open **Manage Jenkins → Tools** (sometimes called **Global Tool Configuration**):
- Configure a JDK installation using Java 17.
- Configure Maven 3.9.x, or install Maven on the Jenkins machine and make it available on PATH.
- Ensure the Jenkins machine can reach GitHub and download Maven dependencies.

### 2. Create the job
1. Select **New Item**.
2. Enter `profile-update-api-build`.
3. Choose **Freestyle project** and click **OK**.
4. Under **Source Code Management**, select **Git**.
5. Paste your GitHub repository URL and choose credentials if the repository is private.
6. Under **Build Steps**, choose **Invoke top-level Maven targets** (if the Maven integration is installed). Select your Maven installation and enter:
   `clean test package`

   If the Maven build step is unavailable, add **Execute shell** on Linux/macOS with `mvn clean test package`, or **Execute Windows batch command** on Windows with `mvn clean test package`.
7. Under **Post-build Actions**, choose **Archive the artifacts** and enter:
   `target/**`
8. Save, then click **Build Now**.
9. Open the build's **Console Output**. A successful build should show tests passing and `BUILD SUCCESS`. Open **Artifacts** on the build page to download the archived files.

Note: this project uses Maven and the standard `target/` output folder. The generated runnable JAR is `target/profile-update-api.jar`.

## Build and run with Docker (optional)

Build the image:
```bash
docker build -t profile-update-api .
```

Run it:
```bash
docker run --rm -p 8080:8080 profile-update-api
```

Then test `http://localhost:8080/api/profile` using the PUT examples above.

## Project structure

```text
profile-update-api/
├── Dockerfile
├── pom.xml
├── README.md
└── src/
    ├── main/java/com/example/profile/
    │   ├── ProfileUpdateApiApplication.java
    │   ├── controller/ProfileController.java
    │   ├── dto/ProfileUpdateRequest.java
    │   ├── dto/ProfileUpdateResponse.java
    │   └── exception/GlobalExceptionHandler.java
    └── test/java/com/example/profile/
        └── ProfileControllerTest.java
```

## Important note
This demo returns the validated values but does not save them to a database. Add a database and repository layer if persistent profile storage is required.
