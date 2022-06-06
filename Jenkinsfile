pipeline {
  agent any

  stages {

    stage('Build Artifact - Maven') {
      steps {
        sh "mvn clean package -DskipTests=true"
        archive 'target/*.jar'
      }
    }

    stage('Unit Tests - JUnit and JaCoCo') {
      steps {
        sh "mvn test"
      }
    }

    stage('Mutation Tests - PIT') {
      steps {
        sh "mvn org.pitest:pitest-maven:mutationCoverage"
      }
    }

  //  stage('SonarQube - SAST') {
  //    steps {
  //      sh "mvn sonar:sonar -Dsonar.projectKey=jenkins-application -Dsonar.host.url=http://192.168.15.18:9000 -Dsonar.login=feb2cf1e5aaa10aad3c2e6a3ff739b35aeb2aa72"
  //    }
  //  }  
   // stage('Vulnerability Scan - Docker ') {
   //   steps {
   //     sh "mvn dependency-check:check"
   //   }
   // }

   // stage('TRIVY Scan2 - Docker2 ') {
   //   steps {
   //     sh "bash trivy-docker-image-scan.sh"
   //   }
   // }

   stage('Vulnerability Scan - Docker') {
      steps {
        parallel(
          "Dependency Scan": {
            sh "mvn dependency-check:check"
          },
          "Trivy Scan": {
            sh "bash trivy-docker-image-scan.sh"
          }
        )
      }
    }
    
    stage('Docker Build and Push') {
      steps {
        withDockerRegistry([credentialsId: "docker-hub", url: ""]) {
          sh 'printenv'
          sh 'docker build -t rcenerini/numeric-app:""$GIT_COMMIT"" .'
          sh 'docker push rcenerini/numeric-app:""$GIT_COMMIT""'
        }
      }
    }

    stage('Kubernetes Deployment - DEV') {
      steps {
        withKubeConfig([credentialsId: 'kubeconfig']) {
          sh "sed -i 's#replace#rcenerini/numeric-app:${GIT_COMMIT}#g' k8s_deployment_service.yaml"
          sh "kubectl apply -f k8s_deployment_service.yaml"
        }
      }
    }
  }
  post {
        always {
          junit 'target/surefire-reports/*.xml'
          jacoco execPattern: 'target/jacoco.exec'
          pitmutation mutationStatsFile: '**/target/pit-reports/**/mutations.xml'
          dependencyCheckPublisher pattern: 'target/dependency-check-report.xml'
        }
      }
      // success {

      //}

      // failed {

      //}

}