# Comprehensive Jenkins CI/CD FAQ

## Basics and Setup

### 1. What is Jenkins and how does it support CI/CD?
Jenkins is an open-source automation server that enables developers to build, test, and deploy their applications. It supports Continuous Integration (CI) by automatically building and testing code changes when they're committed to the repository, detecting errors early. For Continuous Delivery/Deployment (CD), Jenkins automates the process of delivering applications to various environments, ensuring reliable and consistent deployments. Its extensible architecture with over 1,800 plugins allows integration with virtually any tool in the software development lifecycle.

### 2. How do I install Jenkins on different operating systems?

**Linux (Debian/Ubuntu)**:
```bash
# Add repository key
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
# Add the repository to sources
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
# Update and install
sudo apt-get update
sudo apt-get install jenkins
```

**Red Hat/CentOS**:
```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
sudo yum install jenkins
```

**macOS** (using Homebrew):
```bash
brew install jenkins-lts
```

**Windows**:
- Download the installer (.msi file) from Jenkins' website
- Run the installer and follow the wizard
- Access Jenkins through http://localhost:8080/

**Docker**:
```bash
docker run -d -p 8080:8080 -p 50000:50000 jenkins/jenkins:lts
```

### 3. What are the minimum system requirements for Jenkins?
- 256MB RAM (minimum), 1GB+ recommended
- 10GB disk space (minimum), more for bigger installations
- Java 11 or newer (OpenJDK or Oracle JDK)
- Processor: any modern CPU

Requirements increase based on build complexity, number of concurrent builds, and plugins used.

### 4. How do I secure my Jenkins installation?
- Set up Jenkins behind a firewall or reverse proxy
- Use HTTPS with a valid SSL certificate
- Implement proper authentication (LDAP, OAuth, etc.)
- Configure role-based access control
- Keep Jenkins and plugins updated
- Use credential management for secrets
- Enable CSRF protection
- Set up audit logging
- Disable agent-to-master access control (if not needed)
- Regularly backup Jenkins configuration
- Run security scans on your Jenkins instance

### 5. What's the difference between Jenkins and other CI/CD tools like GitLab CI, CircleCI, or GitHub Actions?

| Tool | Hosting | Configuration | Strengths | Limitations |
|------|---------|---------------|-----------|-------------|
| **Jenkins** | Self-hosted | Jenkinsfile or UI | Highly customizable, vast plugin ecosystem, mature platform | Requires maintenance, steeper learning curve |
| **GitLab CI** | SaaS or self-hosted | YAML files | Tight GitLab integration, built-in runners, container registry | Best suited for GitLab repositories |
| **CircleCI** | SaaS | YAML files | Easy setup, good parallelization, first-class Docker support | Limited free tier, can be costly for large teams |
| **GitHub Actions** | SaaS | YAML files | Native GitHub integration, marketplace of actions, easy to start | GitHub-centric, relatively new platform |

## Administration

### 6. How do I set up master-agent architecture in Jenkins?
1. **On the master**:
   - Navigate to "Manage Jenkins" > "Manage Nodes and Clouds"
   - Click "New Node"
   - Enter a name and select "Permanent Agent"

2. **Agent Configuration**:
   - Set number of executors
   - Set remote root directory
   - Choose launch method (SSH, JNLP, etc.)
   - For SSH: provide credentials and host
   - For JNLP: Use the provided command on the agent machine

3. **Different Connection Methods**:
   - SSH: Best for Unix/Linux agents
   - JNLP: Works across platforms, including Windows
   - Docker: Launch agents as containers
   - Kubernetes: Dynamic agent provisioning in K8s clusters

4. **Verifying Connection**:
   - Check log output on the master
   - Verify agent appears in the node list and shows as online

### 7. What's the best backup strategy for Jenkins?
- **What to back up**:
  - $JENKINS_HOME directory (contains all configurations and jobs)
  - Plugin binaries and configurations
  - Global system configurations
  - Job configurations and build history (if needed)
  
- **Backup methods**:
  - Use ThinBackup or Backup plugins
  - Set up periodic file system backups (rsync, etc.)
  - Store configurations in Git using Configuration as Code
  - Use cloud provider snapshots for Jenkins servers

- **Best practices**:
  - Automate backups
  - Store backups offsite
  - Test restore procedures regularly
  - Consider incremental backups for large installations
  - Set appropriate retention policies

### 8. How do I upgrade Jenkins to a newer version?

**War file upgrade**:
1. Download the latest Jenkins WAR
2. Stop Jenkins service
3. Back up $JENKINS_HOME
4. Replace the old WAR with the new one
5. Start Jenkins service

**Package managers**:
```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install jenkins

# CentOS/RHEL
sudo yum update jenkins

# macOS
brew upgrade jenkins-lts
```

**Docker**:
```bash
# Pull the latest image
docker pull jenkins/jenkins:lts
# Stop the current container
docker stop jenkins
# Remove the current container (keep volumes)
docker rm jenkins
# Start a new container with the updated image
docker run -d -p 8080:8080 -p 50000:50000 -v jenkins_home:/var/jenkins_home --name jenkins jenkins/jenkins:lts
```

**Best practices**:
- Always back up before upgrading
- Review update notes for breaking changes
- Test upgrades in non-production environment first
- Consider using LTS versions for stability

### 9. How do I troubleshoot common Jenkins startup issues?

**Common issues and solutions**:

- **Jenkins won't start - Java issues**:
  - Check Java version compatibility: `java -version`
  - Verify JAVA_HOME environment variable
  - Review startup logs for JVM errors

- **Port conflicts**:
  - Check if port 8080 is already in use: `netstat -tuln | grep 8080`
  - Change Jenkins port in `jenkins.xml` or by setting `--httpPort` parameter

- **Permission problems**:
  - Ensure Jenkins user has access to $JENKINS_HOME
  - Check logs for permission denied errors
  - Run `chown -R jenkins:jenkins $JENKINS_HOME`

- **Out of memory**:
  - Increase heap size by setting `JENKINS_JAVA_OPTIONS="-Xmx2g"`
  - Check for memory leaks in plugins

- **Plugin initialization failures**:
  - Start Jenkins in safe mode: `java -jar jenkins.war --safe-mode`
  - Disable problematic plugins

**Key log locations**:
- Linux: `/var/log/jenkins/jenkins.log`
- Windows: `%JENKINS_HOME%\jenkins.log`
- Console output when running directly

### 10. What are Jenkins Blue Ocean and Jenkins X?

**Jenkins Blue Ocean**:
- A modern UI redesign for Jenkins focused on pipeline visualization
- Provides an intuitive, visual representation of pipelines
- Better user experience with simplified pipeline creation
- Built-in Git integration
- Enhanced pipeline debugging
- Compatible with existing Jenkins installations as a plugin

**Jenkins X**:
- A separate project for Kubernetes-native CI/CD
- Automates CI/CD for cloud-native applications on Kubernetes
- Features GitOps workflows and preview environments
- Includes built-in DevOps best practices
- Supports multiple Git providers and Kubernetes clusters
- Uses Tekton for pipeline execution
- Not a UI for standard Jenkins but a separate platform

## Pipeline Creation and Management

### 11. What's the difference between Declarative and Scripted pipelines?

**Declarative Pipeline**:
- Uses a more structured, opinionated syntax
- Starts with `pipeline` block
- Pre-defined sections like `agent`, `stages`, `post`
- Limited flexibility but easier to learn
- Better error reporting and validation
- Example:

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean install'
            }
        }
    }
}
```

**Scripted Pipeline**:
- Based on Groovy scripting
- Starts with `node` block
- Highly flexible and programmatic
- Steeper learning curve
- Example:

```groovy
node {
    stage('Build') {
        sh 'mvn clean install'
    }
}
```

Key differences include syntax structure, flexibility vs. simplicity, error handling, and available features.

### 12. How do I create a basic Jenkinsfile?

A basic Jenkinsfile for a Java application:

```groovy
pipeline {
    agent any
    
    tools {
        maven 'Maven 3.8.6'
        jdk 'JDK 17'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
                junit '**/target/surefire-reports/*.xml'
            }
        }
        
        stage('Package') {
            steps {
                sh 'mvn package'
                archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            echo 'Build successful!'
        }
        failure {
            echo 'Build failed!'
        }
    }
}
```

Essential components:
- `agent`: Where to execute the pipeline
- `tools`: Required tools like Maven, JDK
- `stages`: Collection of stages to execute
- `steps`: Actual commands to run
- `post`: Actions to run after pipeline completion

### 13. How do I implement multi-branch pipelines?

1. **Create a multi-branch pipeline**:
   - In Jenkins, click "New Item"
   - Select "Multibranch Pipeline"
   - Configure source code management (typically Git)
   - Specify branch sources (GitHub, GitLab, etc.)
   - Set branch discovery strategies
   - Add credentials if needed
   - Set scan triggers

2. **Branch Source Configuration**:
   - Repository URL
   - Credentials for authentication
   - Behaviors (discover branches, PRs, tags)
   - Build strategies (regular branches, tags, etc.)

3. **Pipeline Configuration**:
   - Add a Jenkinsfile to your repository root
   - Each branch with a Jenkinsfile will be built automatically
   - Branch-specific logic can be added with conditionals

4. **Example Jenkinsfile with branch conditions**:

```groovy
pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
        }
        
        stage('Deploy to Staging') {
            when {
                branch 'develop'
            }
            steps {
                echo 'Deploying to staging environment'
                // Deploy commands
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                echo 'Deploying to production environment'
                // Production deploy commands
            }
        }
    }
}
```

### 14. What are shared libraries in Jenkins and how do I use them?

Shared libraries allow common pipeline code to be reused across multiple projects.

**Setting up a shared library**:
1. Create a Git repository with the structure:
   ```
   (root)
   +- src                     # Groovy source files
   |   +- org/example/Utils.groovy  # package path
   +- vars                    # Global variables/functions
   |   +- buildJava.groovy    # for global 'buildJava' step
   |   +- buildJava.txt       # documentation
   +- resources               # Resource files
   ```

2. **Configure in Jenkins**:
   - Go to "Manage Jenkins" > "Configure System"
   - Under "Global Pipeline Libraries" add your library
   - Name: "my-shared-lib"
   - Default version: "main" (branch or tag)
   - Retrieval method: modern SCM or legacy SCM
   - Source code management: Git (provide repository URL)

3. **Using in Jenkinsfile**:
   ```groovy
   @Library('my-shared-lib') _
   
   pipeline {
       agent any
       stages {
           stage('Build') {
               steps {
                   // Use a function from vars/buildJava.groovy
                   buildJava()
                   
                   // Or use a class from src/
                   script {
                       def utils = new org.example.Utils()
                       utils.someMethod()
                   }
               }
           }
       }
   }
   ```

4. **Example shared function** (`vars/buildJava.groovy`):
   ```groovy
   def call(String mavenVersion = 'default') {
       sh "mvn -version:${mavenVersion} clean install"
   }
   ```

### 15. How do I implement parallel stages in a Jenkins pipeline?

**Basic parallel execution**:
```groovy
pipeline {
    agent any
    stages {
        stage('Parallel Tests') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'mvn test'
                    }
                }
                stage('Integration Tests') {
                    steps {
                        sh 'mvn verify'
                    }
                }
                stage('Security Scans') {
                    steps {
                        sh './run-security-scan.sh'
                    }
                }
            }
        }
    }
}
```

**Advanced parallel execution with different agents**:
```groovy
pipeline {
    agent none
    stages {
        stage('Parallel Build and Test') {
            parallel {
                stage('Build on Linux') {
                    agent {
                        label 'linux'
                    }
                    steps {
                        sh 'make build'
                    }
                }
                stage('Build on Windows') {
                    agent {
                        label 'windows'
                    }
                    steps {
                        bat 'build.cmd'
                    }
                }
            }
        }
    }
}
```

**Dynamic parallelism**:
```groovy
pipeline {
    agent any
    stages {
        stage('Dynamic Parallel Tests') {
            steps {
                script {
                    def testFolders = sh(script: 'ls -d test-*', returnStdout: true).trim().split('\n')
                    def parallelStages = [:]
                    
                    for (folder in testFolders) {
                        def currentFolder = folder // Important for closure
                        parallelStages["Test ${currentFolder}"] = {
                            sh "cd ${currentFolder} && npm test"
                        }
                    }
                    
                    parallel parallelStages
                }
            }
        }
    }
}
```

## Integration

### 16. How do I integrate Jenkins with Git repositories?

**Basic Git Integration**:
1. Install Git plugin (usually pre-installed)
2. In Jenkins job configuration:
   - Under Source Code Management, select Git
   - Enter repository URL
   - Add credentials if needed
   - Specify branch to build

**Advanced Configuration**:
- Specify Git clone options (shallow clone, timeout, etc.)
- Configure checkout behavior (clean workspace, etc.)
- Set up merge options
- Configure submodule options

**Pipeline Git Integration**:
```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/organization/repo.git',
                    credentialsId: 'github-credentials'
            }
        }
    }
}
```

**Multiple Repositories**:
```groovy
pipeline {
    agent any
    stages {
        stage('Checkout') {
            steps {
                dir('app') {
                    git url: 'https://github.com/org/app.git'
                }
                dir('lib') {
                    git url: 'https://github.com/org/lib.git'
                }
            }
        }
    }
}
```

### 17. How do I set up webhook triggers from GitHub/GitLab/Bitbucket?

**GitHub Webhooks**:
1. **Jenkins Setup**:
   - Install GitHub Integration plugin
   - Set up Jenkins URL in "Manage Jenkins" > "Configure System"
   - For each job, enable "GitHub hook trigger for GITScm polling"

2. **GitHub Configuration**:
   - Go to repository settings > Webhooks
   - Add webhook with Payload URL: `https://[JENKINS_URL]/github-webhook/`
   - Select content type: `application/json`
   - Choose events: Push, Pull Requests (as needed)
   - Save webhook

**GitLab Webhooks**:
1. **Jenkins Setup**:
   - Install GitLab plugin
   - Configure GitLab connection in "Manage Jenkins" > "Configure System"
   - In job, enable "Build when a change is pushed to GitLab"
   - Copy the displayed webhook URL

2. **GitLab Configuration**:
   - Go to project settings > Webhooks
   - Enter the Jenkins webhook URL
   - Select desired triggers (Push, Merge Requests)
   - Add webhook

**Bitbucket Webhooks**:
1. **Jenkins Setup**:
   - Install Bitbucket plugin
   - In job configuration, enable "Build when a change is pushed to Bitbucket"

2. **Bitbucket Configuration**:
   - Go to repository settings > Webhooks
   - Add webhook
   - Enter Jenkins URL: `https://[JENKINS_URL]/bitbucket-hook/`
   - Select events (Repository Push, Pull Request)
   - Save webhook

### 18. How do I integrate Jenkins with container platforms like Docker and Kubernetes?

**Docker Integration**:

1. **Prerequisites**:
   - Install Docker on Jenkins server
   - Add Jenkins user to Docker group: `sudo usermod -aG docker jenkins`
   - Install Docker Pipeline plugin

2. **Using Docker in Pipeline**:
```groovy
pipeline {
    agent {
        docker {
            image 'maven:3.8-openjdk-17'
        }
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
    }
}
```

3. **Building and Publishing Docker Images**:
```groovy
pipeline {
    agent any
    stages {
        stage('Build Image') {
            steps {
                script {
                    dockerImage = docker.build("myapp:${env.BUILD_ID}")
                }
            }
        }
        stage('Push Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.example.com', 'registry-credentials') {
                        dockerImage.push()
                        dockerImage.push('latest')
                    }
                }
            }
        }
    }
}
```

**Kubernetes Integration**:

1. **Prerequisites**:
   - Install Kubernetes plugin
   - Configure Kubernetes cloud in "Manage Jenkins" > "Manage Nodes and Clouds"

2. **Dynamic Jenkins Agents**:
```groovy
pipeline {
    agent {
        kubernetes {
            yaml """
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: maven
    image: maven:3.8-openjdk-17
    command:
    - cat
    tty: true
  - name: docker
    image: docker:19.03.1
    command:
    - cat
    tty: true
    volumeMounts:
    - name: docker-sock
      mountPath: /var/run/docker.sock
  volumes:
  - name: docker-sock
    hostPath:
      path: /var/run/docker.sock
"""
        }
    }
    stages {
        stage('Build') {
            steps {
                container('maven') {
                    sh 'mvn clean package'
                }
            }
        }
        stage('Docker Build') {
            steps {
                container('docker') {
                    sh 'docker build -t myapp:latest .'
                }
            }
        }
    }
}
```

3. **Deploying to Kubernetes**:
```groovy
pipeline {
    agent any
    stages {
        stage('Deploy') {
            steps {
                withKubeConfig([credentialsId: 'k8s-credentials']) {
                    sh 'kubectl apply -f deployment.yaml'
                }
            }
        }
    }
}
```

### 19. How do I implement Jenkins integration with artifact repositories?

**Maven with Nexus/Artifactory**:

1. **Configure Maven settings**:
   - Create `settings.xml` in Jenkins
   - Add server credentials for repository access

2. **Upload artifacts in pipeline**:
```groovy
pipeline {
    agent {
        docker {
            image 'maven:3.8-openjdk-17'
        }
    }
    stages {
        stage('Build and Publish') {
            steps {
                sh 'mvn clean deploy -s settings.xml'
            }
        }
    }
}
```

**Using Artifactory plugin**:

1. **Configure Artifactory server** in "Manage Jenkins" > "Configure System"

2. **Upload with Artifactory plugin**:
```groovy
pipeline {
    agent any
    tools {
        maven 'Maven 3.8'
    }
    stages {
        stage('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Upload') {
            steps {
                rtUpload (
                    serverId: 'artifactory-server',
                    spec: '''{
                        "files": [
                            {
                                "pattern": "target/*.jar",
                                "target": "libs-release-local/myapp/"
                            }
                        ]
                    }'''
                )
            }
        }
    }
}
```

**Docker Registry Integration**:

```groovy
pipeline {
    agent any
    stages {
        stage('Build and Push') {
            steps {
                script {
                    docker.build("myapp:${env.BUILD_ID}")
                    docker.withRegistry('https://registry.example.com', 'registry-credentials') {
                        docker.image("myapp:${env.BUILD_ID}").push()
                    }
                }
            }
        }
    }
}
```

**Generic Artifact Upload (like AWS S3)**:

```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
        stage('Upload to S3') {
            steps {
                withAWS(credentials: 'aws-credentials', region: 'us-east-1') {
                    s3Upload(file:'build/', bucket:'my-app-artifacts', path:'releases/${env.BUILD_ID}/')
                }
            }
        }
    }
}
```

### 20. How do I integrate testing frameworks with Jenkins?

**JUnit Integration**:
```groovy
pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
    }
}
```

**TestNG Integration**:
```groovy
pipeline {
    agent any
    stages {
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    step([$class: 'Publisher', reportFilenamePattern: '**/testng-results.xml'])
                }
            }
        }
    }
}
```

**JavaScript Testing (Jest/Mocha)**:
```groovy
pipeline {
    agent {
        docker {
            image 'node:16'
        }
    }
    stages {
        stage('Test') {
            steps {
                sh 'npm install'
                sh 'npm test'
            }
            post {
                always {
                    junit 'junit-reports/*.xml'
                }
            }
        }
    }
}
```

**Selenium Integration**:
```groovy
pipeline {
    agent any
    stages {
        stage('UI Tests') {
            steps {
                sh 'mvn clean test -Dtest=SeleniumTestSuite'
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'
                }
            }
        }
    }
}
```

**Code Coverage (JaCoCo)**:
```groovy
pipeline {
    agent any
    stages {
        stage('Test with Coverage') {
            steps {
                sh 'mvn clean test jacoco:report'
            }
            post {
                success {
                    jacoco(
                        execPattern: 'target/jacoco.exec',
                        classPattern: 'target/classes',
                        sourcePattern: 'src/main/java',
                        exclusionPattern: 'src/test/*'
                    )
                }
            }
        }
    }
}
```

## Security

### 21. How do I implement role-based access control in Jenkins?

1. **Install required plugins**:
   - Role-based Authorization Strategy plugin

2. **Enable role-based strategy**:
   - Go to "Manage Jenkins" > "Configure Global Security"
   - Under "Authorization", select "Role-Based Strategy"
   - Save

3. **Configure roles**:
   - Go to "Manage Jenkins" > "Manage and Assign Roles"
   - Click "Manage Roles"

4. **Create roles**:
   - **Global roles**: Apply across Jenkins
     - Admin: Overall/Administer
     - Developer: Read, Job/Build
     - Viewer: Read-only access
   
   - **Project roles**: Apply to specific jobs/folders
     - Pattern: `project-a.*` (regex for projects starting with "project-a")
     - ProjectA-Developer: Job Build, Cancel, Configure, Read
   
   - **Node roles**: Apply to specific agents
     - Pattern: `production-.*` (regex for production nodes)
     - ProductionAdmin: Configure, Connect, Create, Delete

5. **Assign roles to users and groups**:
   - Go to "Assign Roles" tab
   - Add users/groups to appropriate roles

6. **Best practices**:
   - Follow principle of least privilege
   - Use project-specific roles for team isolation
   - Regular audit of permissions
   - Use groups when possible (e.g., LDAP groups)

### 22. How do I manage secrets and credentials securely in Jenkins?

1. **Jenkins Credentials Plugin**:
   - Go to "Manage Jenkins" > "Manage Credentials"
   - Choose appropriate domain
   - Add credentials of different types:
     - Username with password
     - SSH Username with private key
     - Secret text
     - Secret file
     - Certificate

2. **Using credentials in Jenkinsfile**:
   ```groovy
   pipeline {
       agent any
       stages {
           stage('Deploy') {
               steps {
                   withCredentials([
                       string(credentialsId: 'api-key', variable: 'API_KEY'),
                       usernamePassword(credentialsId: 'db-creds', usernameVariable: 'DB_USER', passwordVariable: 'DB_PASSWORD')
                   ]) {
                       sh 'deploy.sh ${API_KEY} ${DB_USER} ${DB_PASSWORD}'
                   }
               }
           }
       }
   }
   ```

3. **Secret masking**:
   - Jenkins automatically masks secrets in build logs
   - Avoid echoing secrets to the console

4. **Credential rotation**:
   - Implement regular credential rotation
   - Update Jenkins credentials without changing pipelines

5. **External secret management integration**:
   - HashiCorp Vault plugin
   - AWS Secrets Manager plugin
   - Azure Key Vault plugin

6. **Vault integration example**:
   ```groovy
   pipeline {
       agent any
       stages {
           stage('Get Secrets') {
               steps {
                   withVault(
                       configuration: [
                           timeout: 60,
                           vaultUrl: 'https://vault.example.com:8200',
                           vaultCredentialId: 'vault-app-role'
                       ],
                       vaultSecrets: [
                           [path: 'secret/myapp/config', secretValues: [
                               [envVar: 'API_KEY', vaultKey: 'api_key'],
                               [envVar: 'DB_PASSWORD', vaultKey: 'db_password']
                           ]]
                       ]
                   ) {
                       sh 'deploy.sh ${API_KEY} ${DB_PASSWORD}'
                   }
               }
           }
       }
   }
   ```

### 23. What are the best practices for securing Jenkins pipelines?

1. **Pipeline security principles**:
   - Validate all inputs
   - Minimize plugin usage to necessary ones
   - Keep Jenkins and plugins updated
   - Use declarative pipelines when possible
   - Enforce code reviews for pipeline changes

2. **Script Security**:
   - Use script approval for custom scripts
   - Avoid approving potentially dangerous methods
   - Use trusted libraries for common tasks
   - Implement Groovy sandbox for scripts

3. **Access Control**:
   - Implement project-based security
   - Restrict who can define pipelines
   - Use credential binding instead of environment variables
   - Separate credentials by environment

4. **Build Environment Security**:
   - Use ephemeral build agents when possible
   - Isolate builds with containers or VMs
   - Scan artifacts for vulnerabilities
   - Clean workspaces after builds

5. **Example of secure script practices**:
   ```groovy
   // Bad practice
   def command = "curl ${params.URL}" // Potential command injection
   sh command
   
   // Good practice
   if (params.URL =~ /^https:\/\/api\.company\.com\//) {
       sh "curl ${params.URL}"
   } else {
       error "Invalid URL. Must be from api.company.com domain"
   }
   ```

6. **Securing input parameters**:
   ```groovy
   pipeline {
       agent any
       parameters {
           choice(name: 'ENVIRONMENT', choices: ['dev', 'test', 'staging', 'prod'], description: 'Deployment Environment')
           string(name: 'VERSION', defaultValue: '1.0.0', description: 'Version to deploy')
       }
       stages {
           stage('Deploy') {
               steps {
                   script {
                       // Validate input parameters
                       if (!(params.VERSION =~ /^\d+\.\d+\.\d+$/)) {
                           error "Invalid version format. Must be semantic version (e.g., 1.0.0)"
                       }
                       
                       // Proceed with deployment
                       deploy(params.ENVIRONMENT, params.VERSION)
                   }
               }
           }
       }
   }
   ```

### 24. How do I implement audit logging in Jenkins?

1. **Built-in logging**:
   - Enable Jenkins logging in "Manage Jenkins" > "System Log"
   - Configure log levels for components
   - Access logs at "Manage Jenkins" > "System Log"

2. **Audit Trail plugin**:
   - Install "Audit Trail" plugin
   - Configure in "Manage Jenkins" > "Configure System"
   - Options for logging to file or syslog
   - Configure log rotation and retention

3. **What actions to audit**:
   - Job creation, modification, deletion
   - Build starts and completions
   - Configuration changes
   - User authentication events
   - Permission changes
   - System configuration updates

4. **Forwarding logs to SIEM systems**:
   - Configure syslog forwarding
   - Send logs to Spl
