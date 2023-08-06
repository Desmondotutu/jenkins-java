pipeline {
 agent any
   environment{
    mvn = tool 'maven3'
    DependencyCheck = tool 'DP-Check'
    DOCKER_IMAGE = "desmondo1/express:${BUILD_NUMBER}"
    }
    stages {
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
        sh "${DependencyCheck}/bin/dependency-check.sh --scan . dependencyCheckPublisher pattern: '**/dependency-check-report.xml'"
      }
    }
    
    stage('Static Code Analysis') {

      steps {
        withSonarQubeEnv('SonarqubeServer10') {
          sh "${mvn}/bin/mvn clean verify sonar:sonar -Dsonar.projectKey=jenkins-java"
        }
      }
    }
    stage('Build Docker Image') {
      steps {
        script {
            sh "docker build -t ${DOCKER_IMAGE} ."
        }
      }
    }
     stage('Scan Docker Image') {
        steps {
         script{
           sh "trivy image ${DOCKER_IMAGE}"
            }
        }
        }
     stage('Push Docker Image') {
      steps {
           script{
             withDockerRegistry(credentialsId: 'dockerHubCredentials'){
             sh "docker push ${DOCKER_IMAGE}"
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
