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
