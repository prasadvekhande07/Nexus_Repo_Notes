# Nexus_Repo_Notes

---

## Artifact Repository

### What is an Artifact Repository?
- **Artifacts**: Applications built into a single, sharable, and movable file.
- **Artifact Formats**: JAR, WAR, ZIP, TAR, etc.
- **Artifact Repository**: A storage solution for these artifacts that supports specific formats.

### Artifact Repository Manager
- **Examples**: Nexus, JFrog Artifactory.
- **Purpose**: Manages different types of artifact repositories in an organization.
- **Key Features**:
  - **Host Own Repositories**: Manage internal repositories.
  - **Proxy Repository**: Proxy both company-internal and public repositories.
  - **Multi-Format Support**: Support for various artifact formats (e.g., JAR, Docker, etc.).
  - **Access Management**: Integrate with LDAP to configure access control.
  - **REST API**: Flexible and powerful REST API for automation and integration with CI/CD tools.
  - **Backup and Restore**: Ensure data safety with backup and restore capabilities.
  - **Cleanup Policies**: Automatically delete files that match certain conditions.
  - **Search Functionality**: Search across projects and repositories.
  - **User Token Support**: For system user authentication.

### Setting Up Nexus Repository Manager

#### Download and Extract Nexus:
```bash
# Verify Java installation
$ java -version

# Navigate to the desired directory
$ cd /opt

# Download Nexus Repository Manager
$ wget https://download.sonatype.com/nexus/3/latest-unix.tar.gz

# Extract the downloaded file
$ tar -zxvf latest-unix.tar.gz

# List the contents
$ ls
```

#### Nexus Folder Structure:
- **Nexus Folder**: Contains runtime and application files.
  - Example:
    ```
    latest-unix.tar.gz nexus-3.28.1-01 sonatype-work
    ```
  - Content:
    ```
    nexus-3.28.1-01/
    ├── NOTICE.txt
    ├── OSS-LICENSE.txt
    ├── PRO-LICENSE.txt
    ├── bin
    ├── deploy
    ├── etc
    ├── lib
    └── public
    ```
- **Sonatype-work Folder**: Contains custom configuration and data for Nexus.
  - Example:
    ```
    sonatype-work/nexus3/
    ├── clean_cache
    ├── log
    ├── orient
    └── tmp
    ```
  - **Usage**: Backup all Nexus data, logs, and metadata.

#### Starting Nexus:

1. **Create a Nexus User**:
    ```bash
    # Create a new user for Nexus
    $ adduser nexus
    
    # Assign ownership to nexus user
    $ chown -R nexus:nexus nexus-3.28.1-01
    $ chown -R nexus:nexus sonatype-work
    ```

2. **Configure Nexus to Run as Nexus User**:
    ```bash
    # Edit the nexus.rc file to set the user
    $ vim nexus-3.28.1-01/bin/nexus.rc
    
    # Uncomment and set: run_as_user="nexus"
    ```

3. **Start Nexus**:
    ```bash
    # Switch to nexus user
    $ su - nexus
    
    # Start the Nexus service
    $ /opt/nexus-3.28.1-01/bin/nexus start
    
    # Identify the process ID
    $ ps aux | grep nexus
    
    # Check the port on which the service is running
    $ netstat -lnpt
    ```

4. **Access Nexus**:
    - Use `<ip_address>:8081` in a browser to access the Nexus Repository Manager.

---


---

## Repository Types in Nexus

### 1. Proxy Repository
- **Purpose**: Download dependencies from remote public or private repositories.
- **Usage**: Acts as a cache for external repositories, reducing external network dependency and improving build performance.

### 2. Hosted Repository
- **Purpose**: Store artifacts that are deployed within the organization.
- **Types**:
  - **Snapshot Repository**:
    - Stores artifacts deployed from feature or development branches.
    - Used during the development cycle for continuous integration.
  - **Release Repository**:
    - Stores production-ready artifacts.
    - Typically deployed from the master branch or release tags.
    - Ensures that only stable and tested versions of artifacts are available for production use.

### 3. Group Repository
- **Purpose**: Group multiple repositories (both hosted and proxy) under a single URL.
- **Usage**: Simplifies access to multiple repositories by combining them into a single repository endpoint.

---

This information expands on the different types of repositories managed by a Repository Manager like Nexus."


"Here’s the combined and structured version of your notes for configuring and uploading JAR files to a Nexus repository using both Maven and Gradle projects. You can use this in your `README.md` file:

---

## Uploading JAR Files to Nexus Repository (Maven & Gradle)

### Overview
This guide covers the steps to upload JAR files from both Maven and Gradle projects to a Nexus repository, including how to configure both tools to connect to Nexus and use specific credentials.

### 1. Configuring Gradle Project for Nexus

#### a. Set Project Name in `settings.gradle`
```groovy
$ vi settings.gradle

rootProject.name = 'myapp'
```

#### b. Configure `build.gradle` for Publishing
```groovy
$ vi build.gradle

group 'com.example'
version '1.0-SNAPSHOT'

apply plugin: 'maven-publish'  # Enables 'gradlew publish' command

publishing {
    publications {
        maven(MavenPublication) {
            artifact("build/libs/my-app-$version.jar") {
                extension 'jar'
            }
        }
    }
    repositories {
        maven {
            name 'nexus'
            url "http://[your_nexus_ip]:[your_nexus_port]/repository/maven-snapshots/"
            credentials {
                username project.repoUser
                password project.repoPasswd
            }
        }
    }
}
```
- **Explanation**:
  - **`apply plugin: 'maven-publish'`**: Enables publishing capabilities.
  - **`artifact("build/libs/my-app-$version.jar")`**: Specifies the JAR file to upload.
  - **Repository Configuration**: Includes Nexus repository URL and credentials.
  - We need to manually create this user and assign roles to access maven snapshot repository to upload the artifacts for this follow this step
    > ### 7. Creating a Nexus User with Specific Roles (search with ctrl + f)
 

#### c. Store Nexus Credentials Securely in `gradle.properties`
```groovy
$ vi gradle.properties

repoUser = Prasad
repoPasswd = Mypassword
```
- **Note**: Keep the `gradle.properties` file secure and do not commit it to the repository.

### 2. Build and Upload the JAR File (Gradle)

#### a. Build the Application
```bash
$ ./gradlew build
```
- **Output**: Builds the application and generates the JAR file at `build/libs/my-app-1.0-SNAPSHOT.jar`.

#### b. Publish the JAR File to Nexus
```bash
$ ./gradlew publish
```
- **Action**: Uploads the JAR file to the configured Nexus repository.

### 3. Configuring Maven Project for Nexus

#### a. Set Up `pom.xml` for Publishing
```xml
$ vi pom.xml

<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>java-maven-app</artifactId>
    <version>1.0-SNAPSHOT</version>

    <build>
        <pluginManagement>
            <plugins>
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-deploy-plugin</artifactId>
                    <version>2.8.2</version>
                </plugin>
            </plugins>
        </pluginManagement>

        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-deploy-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

    <distributionManagement>
        <snapshotRepository>
            <id>nexus-snapshots</id>
            <url>http://[your_nexus_ip]:[your_nexus_port]/repository/maven-snapshots/</url>
        </snapshotRepository>
    </distributionManagement>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>2.3.4.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.13.1</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>net.logstash.logback</groupId>
            <artifactId>logstash-logback-encoder</artifactId>
            <version>6.4</version>
        </dependency>
    </dependencies>
</project>
```
- **Explanation**:
  - **`maven-deploy-plugin`**: Used to deploy JAR files to Nexus.
  - **`distributionManagement`**: Configures the repository for snapshots with the Nexus URL.

#### b. Configure Nexus Credentials in `settings.xml`
```xml
$ cd ~/.m2

$ vi settings.xml

<settings>
    <servers>
        <server>
            <id>nexus-snapshots</id>
            <username>nana</username>
            <password>xxxxx</password>
        </server>
    </servers>
</settings>
```
- **Note**: Ensure the `id` in `settings.xml` matches the `id` in `pom.xml`.

### 4. Build and Upload the JAR File (Maven)

#### a. Package the Application
```bash
$ mvn package
```
- **Output**: Creates the JAR file in the `target` directory.

#### b. Deploy the JAR File to Nexus
```bash
$ mvn deploy
```
- **Action**: Deploys the JAR file to the Nexus repository.

### 5. Nexus User Configuration

- **Realistic Use Case**:
  - Do not give developers admin credentials for Nexus.
  - Create a specific Nexus user with upload permissions via the Nexus UI.

### 6. Structure in Nexus Repository
After deploying, artifacts will be organized under their respective groups:
- **Maven**: `com.example/java-maven-app`
- **Gradle**: `com.example/my-app`

---

Here's an additional section to your notes that explains how to manually create a user in Nexus and assign roles for accessing the Maven snapshot repository:

---

### 7. Creating a Nexus User with Specific Roles

To securely upload artifacts to your Nexus repository, it's best to create a dedicated user with the necessary permissions rather than using admin credentials.

#### a. Log in to Nexus Repository Manager

1. Open your browser and go to the Nexus Repository Manager URL.
2. Log in using your admin credentials.

#### b. Create a New User

1. **Navigate to**: "Security" > "Users".
2. Click on the "Create user" button.

3. **Fill in the Details**:
   - **Username**: Provide a username (e.g., `upload_user`).
   - **Password**: Set a secure password.
   - **First Name / Last Name / Email**: Optional but recommended for identification.

4. **Assign Roles**:
   - Scroll down to the "Roles" section.
   - Assign roles that allow access to the Maven snapshot repository, such as:
     - **nx-repository-view-maven2-snapshot-*:read**
     - **nx-repository-view-maven2-snapshot-*:write**
     - **nx-repository-view-*-*:add**
     - **nx-repository-view-*-*:browse**
   - Optionally, assign more specific roles depending on your security policy.

5. **Save the User**: Click "Create user" to save.

#### c. Use This User for Maven/Gradle Uploads

- Use the credentials of this newly created user in your `settings.xml` (for Maven) or `gradle.properties` (for Gradle) to upload artifacts.

```xml
<server>
    <id>nexus-snapshots</id>
    <username>upload_user</username>
    <password>secure_password</password>
</server>
```

```groovy
repoUser = upload_user
repoPasswd = secure_password
```

### Benefits

- **Security**: By creating a specific user with limited roles, you minimize the security risks associated with using admin credentials.
- **Access Control**: Assigning roles ensures that the user can only perform actions related to uploading artifacts, nothing more.

---

This section should provide a clear guide on how to create a Nexus user with the appropriate roles for uploading artifacts to the Maven snapshot repository.

This consolidated guide covers both Maven and Gradle projects for uploading JAR files to Nexus, ensuring that you have all the necessary steps in one place."


Here are your notes on Nexus API:

---

## Nexus API - Introduction

### Overview
- Nexus API allows querying the Nexus Repository for various information, such as:
  - **Available Components**: Determine which components are stored in the repository.
  - **Available Repositories**: Identify the different repositories in Nexus.
  - **Available Versions**: Find out which versions of components are available.

### Use Cases
- **CI/CD Pipeline Integration**: 
  - Fetch information on available versions to decide which version to deploy to staging or production environments.
  - Automate the process of selecting and deploying specific versions programmatically, especially useful when pushing multiple artifacts daily.
- **Build Automation**: 
  - Integrate with the Nexus API to automate artifact version management in complex build automation setups.

### Flexibility
- Nexus APIs are flexible and offer several important endpoints for interacting with the repository.

---

### How to Access the REST Endpoint

- **Tools**: Use tools like `curl` or `wget` to execute HTTP requests.
- **Authentication**: Provide the credentials of a Nexus user with the necessary permissions.

#### Example 1: Querying Repositories

```bash
$ curl -u nana:Bib110123 -X GET 'http://167.99.248.163:8081/service/rest/v1/repositories'
```

Response:
```json
{
  "name": "maven-snapshots",
  "format": "maven2",
  "type": "hosted",
  "url": "http://167.99.248.163:8081/repository/maven-snapshots",
  "attributes": {}
}
```

- **Note**: If the user has access to only one repository, only that repository will be shown in the response.

#### Example 2: Listing Components in a Repository

```bash
$ curl -u nana:Bibilo123 -X GET 'http://167.99.248.163:8081/service/rest/v1/components?repository=maven-snapshots'
```

- This command lists all components within the `maven-snapshots` repository using a query parameter. If the user lacks access, it won't throw an error but simply won’t return any data.

#### Example 3: Retrieving a Specific Component by ID

```bash
$ curl -u nana:Bibilo123 -X GET 'http://167.99.248.163:8081/service/rest/v1/components/bWF2ZW4tc25hcHNob3RzOjcxYWZ]YTU®MGUyM2RkZTViMmQ4NGY3ZmE4MDk5TM'
```

- To fetch details of a specific component, append its ID to the components endpoint.

---

### Endpoint Structure

- **Base URL**: `http://167.99.248.163:8081/`
- **Common Endpoint**: `/service/rest/v1/`
- **Examples**:
  - **Repositories**: `/service/rest/v1/repositories`
  - **Components**: `/service/rest/v1/components`

---

This should give you a clear understanding of how to interact with Nexus using its REST API.

Here are your notes on Blob Store in Nexus:

---

## Blob Store - Introduction

### Overview
- **Blob Store**: Nexus Storage used to store all uploaded files, specifically binary files.
- **Storage Options**: Files can be stored locally or in cloud storage.

### File Structure
- Navigate to the Nexus work directory to view blob stores:
  
```bash
$ cd /opt/sonatype-work/nexus3/
$ ls
blobs db etc instances keystores cache elasticsearch generated-bundles karaf.pid lock orient restore-from-backup
```

- **Blobs Directory**: Contains all blob stores.
  
```bash
$ cd blobs
$ ls 
default
```

- **Content Directory**: Holds all the stored data.
  
```bash
$ cd default
$ ls 
9C336132-6A408647-FA393A01-C324F353-CF09AD4B-deletions.index content 9C336132-6A408647-FA393A01-C324F353-CF09AD4B-metrics.properties metadata.properties
```

---

## Blob Store Types

- **Type Field**: Indicates the storage backend type.
  - **File**: File system-based storage (default option).
  - **S3**: Cloud-based storage (used when Nexus is hosted on AWS).

- **State Field**: Represents the current state of the blob store.
  - **Started**: Blob store is running as expected.
  - **Failed**: Indicates a configuration issue, and the store failed to initialize.

- **Blob Count Field**: Shows the number of blobs currently stored.

---

## Creating File-Type Blob Storage

- **Path**: 
  - Must provide an absolute path to the desired file system location.
  - Example: `/opt/sonatype-work/nexus3/blobs/mystore`
  - Ensure the path is fully accessible by the OS user account (Nexus user).

---

## Blob Store - Considerations

- **Immutability**: 
  - Blob stores cannot be modified once created.
  - Blob stores used by a repository cannot be deleted.

- **Planning**:
  - Decide how many blob stores to create and their sizes.
  - Determine which blob stores will be used for which repositories.
  - Estimate the approximate space requirements for each repository.

- **Repository Allocation**:
  - Once a repository is allocated to a blob store, it is permanently allocated.
  - A single repository cannot use multiple blob storages.

---

This guide should help you understand the basics of Blob Store in Nexus and the key considerations when managing it.
