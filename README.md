# 3-Layer-Java-Application
building a 3-layered Java application, setting up Jenkins, and creating a CI/CD pipeline that utilizes Docker containers.

Here’s a step-by-step approach:

Step 1: Setup EC2 Instance
Create an EC2 Instance:

Launch an EC2 instance using an Ubuntu image.
Install necessary packages:
bash
Copy code
sudo apt update
sudo apt install openjdk-11-jdk git maven docker.io -y
Install Jenkins:

Add Jenkins repository key and sources:
bash
Copy code
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins -y
Start Jenkins:
bash
Copy code
sudo systemctl start jenkins
sudo systemctl enable jenkins
Configure Jenkins:

Get the initial password:
bash
Copy code
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
Open the Jenkins URL in the browser http://<EC2-PUBLIC-IP>:8080.
Complete the setup wizard, install suggested plugins, and create an admin user.
Step 2: Create a 3-Layer Java Application
We'll create a simple 3-layer architecture:

Controller Layer: Handles HTTP requests.
Service Layer: Implements business logic.
Repository Layer: Handles database operations (in our case, a simple in-memory operation).
Directory Structure:
bash
Copy code
myapp/
├── src
│   ├── main
│   │   ├── java
│   │   │   └── com
│   │   │       └── example
│   │   │           ├── controller
│   │   │           ├── service
│   │   │           └── repository
│   │   └── resources
│   └── test
└── pom.xml
Step 2.1: Create Directories
bash
Copy code
mkdir -p myapp/src/main/java/com/example/controller
mkdir -p myapp/src/main/java/com/example/service
mkdir -p myapp/src/main/java/com/example/repository
mkdir -p myapp/src/main/resources
mkdir -p myapp/src/test
Step 2.2: Write the Java Code
Controller: src/main/java/com/example/controller/MyController.java

java
Copy code
package com.example.controller;

import com.example.service.MyService;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class MyController {
    private final MyService myService;

    public MyController(MyService myService) {
        this.myService = myService;
    }

    @GetMapping("/hello")
    public String hello() {
        return myService.getMessage();
    }
}
Service: src/main/java/com/example/service/MyService.java

java
Copy code
package com.example.service;

import com.example.repository.MyRepository;
import org.springframework.stereotype.Service;

@Service
public class MyService {
    private final MyRepository myRepository;

    public MyService(MyRepository myRepository) {
        this.myRepository = myRepository;
    }

    public String getMessage() {
        return myRepository.fetchMessage();
    }
}
Repository: src/main/java/com/example/repository/MyRepository.java

java
Copy code
package com.example.repository;

import org.springframework.stereotype.Repository;

@Repository
public class MyRepository {
    public String fetchMessage() {
        return "Hello from the 3-layered Java application!";
    }
}
pom.xml: This file defines your Java project's dependencies.

xml
Copy code
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>myapp</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!-- Spring Boot Starter Web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <version>2.7.0</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <!-- Spring Boot Maven Plugin -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
Step 3: Setup Git and Push to GitHub
Install Git (if not already installed):

bash
Copy code
sudo apt install git
Initialize a Git repository and push the code to GitHub:

bash
Copy code
cd myapp
git init
git add .
git commit -m "Initial commit of 3-layer Java app"
Create a repository on GitHub:

Go to GitHub and create a new repository.
Push your code to GitHub:
bash
Copy code
git remote add origin https://github.com/yourusername/myapp.git
git branch -M main
git push -u origin main
Step 4: Jenkins Pipeline with Docker
In Jenkins, we'll use a Docker-based pipeline to build and run the Java application.

Install Docker Plugin in Jenkins:

In Jenkins, go to Manage Jenkins > Manage Plugins, and install the Docker Pipeline plugin.
Create Jenkins Pipeline:

In Jenkins, create a new pipeline project.
Add the following pipeline script in Jenkins:
groovy
Copy code
pipeline {
    agent {
        docker {
            image 'maven:3.8.4-jdk-11' // Maven Docker image
            args '-v /root/.m2:/root/.m2' // Cache Maven dependencies
        }
    }
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/yourusername/myapp.git'
            }
        }
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
        stage('Run') {
            steps {
                sh 'mvn spring-boot:run &'
                sleep(20)
                sh 'curl http://localhost:8080/hello'
            }
        }
    }
    post {
        always {
            sh 'docker container prune -f' // Clean up containers
        }
    }
}
Run the Pipeline:

Trigger the Jenkins pipeline, and it will build, test, and run your Java application inside a Docker container.
Step 5: Final Deployment in Jenkins
Once the pipeline is running successfully:

Jenkins will pull the code from GitHub.
Maven will build the application.
The application will run in a Docker container and will be tested with curl.
Step 6: Push to GitHub and Further Modifications
Do you want to make any more modifications before I assist you in pushing to GitHub and setting up the deployment in Jenkins?

Let me know if you'd like me to help further with your Jenkins pipeline or push your application to GitHub!
