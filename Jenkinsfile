pipeline {
 agent any
   environment{
    mvn = tool 'maven3'
    DOCKER_IMAGE = "desmondo1/express:${BUILD_NUMBER}"
    }
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/Desmondotutu/jenkins-java.git'
             }
        }
        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }
        stage('Test') { 
            steps {
                sh 'mvn test' 
            }
        }
      stage('Dependency Check') {

      steps {
        dependencyCheck additionalArguments: ' --scan ./', odcInstallation: 'DP-Check' 
        dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
      }
    }
    
    stage('Static Code Analysis') {

      steps {
        withSonarQubeEnv('sonarServer9.9') {
          sh "${mvn}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=maven-project"
        }
      }
    }
    stage('Build and Push Docker Image') {
      steps {
        script {
            sh 'docker build -t ${DOCKER_IMAGE} .'
            def dockerImage = docker.image("${DOCKER_IMAGE}")
            docker.withRegistry('https://index.docker.io/v1/', "dockerhub_creds") {
                dockerImage.push()
            }
        }
      }
    }
     /*
    stage('Update Deployment File') {
        environment {
            GIT_REPO_NAME = "Jenkins-Zero-To-Hero"
            GIT_USER_NAME = "Desmondotutu"
        }
        steps {
            withCredentials([string(credentialsId: 'github_key', variable: 'GITHUB_TOKEN')]) {
                sh '''
                    git config user.email "dwangki2@gmail.com"
                    git config user.name "Desmondotutu"
                    BUILD_NUMBER=${BUILD_NUMBER}
                    sed -i "s/replaceImageTag/${BUILD_NUMBER}/g" java-maven-sonar-argocd-helm-k8s/spring-boot-app-manifests/deployment.yml
                    git add java-maven-sonar-argocd-helm-k8s/spring-boot-app-manifests/deployment.yml
                    git commit -m "Update deployment image to version ${BUILD_NUMBER}"
                    git push https://${GITHUB_TOKEN}@github.com/${GIT_USER_NAME}/${GIT_REPO_NAME} HEAD:main
                '''
            }
        }
    } */
  }
}
