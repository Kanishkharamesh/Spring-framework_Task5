# Jenkins Pipeline Setup for Spring Framework

This covers setting up a **Jenkins pipeline** to build a Spring Boot project using **Maven** and **Docker**.

---

## 1️⃣ Clone the Git Repository

First, clone the repository and push it to your own GitHub repo:

```sh
# Clone the repository
git clone https://github.com/original-repo/spring-framework.git

# Change to the project directory
cd spring-framework

# Add your GitHub repository as the remote
git remote set-url origin https://github.com/kanishkharamesh/spring-framework.git

# Push the project to your own repo
git push origin main
```

---

## 2️⃣ Install Maven

Install or update Maven on your system:

```sh
# Update package lists
sudo apt update

# Install Maven
sudo apt install maven -y
```

**Check Maven Version:**

```sh
mvn -version
```

If the installed version is outdated, remove it and install the latest version manually:

```sh
# Remove existing Maven
sudo apt remove maven -y

# Download the latest Maven version
cd /opt
sudo wget https://downloads.apache.org/maven/maven-3/3.9.6/binaries/apache-maven-3.9.6-bin.tar.gz

# Extract and move Maven
sudo tar -xvzf apache-maven-3.9.6-bin.tar.gz
sudo mv apache-maven-3.9.6 /opt/maven

# Set up environment variables
echo 'export M2_HOME=/opt/maven' | sudo tee -a /etc/profile.d/maven.sh
echo 'export PATH=$M2_HOME/bin:$PATH' | sudo tee -a /etc/profile.d/maven.sh
source /etc/profile.d/maven.sh
```

Verify installation:

```sh
mvn -version
which mvn
```

If necessary, create a symbolic link:

```sh
sudo ln -s /opt/maven/bin/mvn /usr/bin/mvn
```

---

## 3️⃣ Build the Project with Maven

Navigate to the Jenkins workspace and build the project:

```sh
cd /var/lib/jenkins/workspace/Spring-framework

# Clean and package the project (skipping tests)
mvn clean package -DskipTests
```

---

## 4️⃣ Set Up Jenkins Pipeline

1. Open **Jenkins Dashboard** → Click **New Item** → Select **Pipeline**.
2. Go to **Pipeline** section and add the following script:

```groovy
pipeline {
    agent any

    environment {
        IMAGE_NAME = "docker-user-name/my-app"
        REGISTRY = "docker.io"
        DOCKER_USER = "docker-user-name"
        DOCKER_PASS = "your-docker-password"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git url: 'https://github.com/git-user-name/git-repo-name.git', branch: 'main'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t $IMAGE_NAME:latest ."
                }
            }
        }

        stage('Login to Docker Registry') {
            steps {
                script {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                }
            }
        }

        stage('Push Image to Docker Registry') {
            steps {
                script {
                    sh "docker push $IMAGE_NAME:latest"
                }
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed! Check the logs for errors.'
        }
    }
}
```

**⚠️ Important:** Avoid hardcoding Docker passwords. Instead, use Jenkins credentials.

---

## 5️⃣ Fix Permissions for Jenkins

Ensure Jenkins has the correct permissions:

```sh
sudo chown -R jenkins:jenkins /var/lib/jenkins/workspace/Spring-framework
sudo chmod -R 775 /var/lib/jenkins/workspace/Spring-framework
```

Restart Jenkins to apply changes:

```sh
sudo systemctl restart jenkins
```

---

## 6️⃣ Run and Debug the Pipeline

After setting up everything, go to Jenkins and **trigger the build**.
If there are any errors:

```sh
docker images    # Check if the image exists  
docker ps -a     # Check running containers  
docker logs <container_id>  # View container logs  
```
