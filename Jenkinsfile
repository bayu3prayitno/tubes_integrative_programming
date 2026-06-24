pipeline {
     agent any
     triggers {
          pollSCM('* * * * *')
     }
     stages {
          stage("Compile") {
               steps {
                    sh "chmod +x gradlew"
                    sh "./gradlew compileJava"
               }
          }
          stage("Unit test") {
               steps {
                    sh "./gradlew test"
               }
          }
          stage("Code coverage") {
               steps {
                    sh "./gradlew jacocoTestReport"
                    sh "./gradlew jacocoTestCoverageVerification"
               }
          }
          stage("Static code analysis") {
               steps {
                    sh "./gradlew checkstyleMain"
               }
          }
          stage("Package") {
               steps {
                    sh "./gradlew build"
               }
          }

          stage("Docker build") {
               steps {
                    sh "docker build -t bayutri22/calculator:${BUILD_TIMESTAMP} ."
               }
          }

          stage("Docker push") {
               steps {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                         sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"
                         sh "docker push bayutri22/calculator:${BUILD_TIMESTAMP}"
                    }
               }
          }

          stage("Update version") {
               steps {
                    sh "sed -i 's/{{VERSION}}/${BUILD_TIMESTAMP}/g' deployment.yaml"
               }
          }

          stage("Deploy to staging") {
               steps {
                    sh "kubectl config use-context staging"
                    sh "kubectl apply -f hazelcast.yaml"
                    sh "kubectl apply -f deployment.yaml"
                    sh "kubectl apply -f service.yaml"
               }
          }

          stage("Acceptance test") {
               steps {
                    sh "kubectl rollout status deployment/calculator-deployment --timeout=180s"
                    sh "chmod +x acceptance-test.sh && ./acceptance-test.sh"
               }
          }

          stage("Performance test") {
               steps {
                    sh "chmod +x performance-test.sh && ./performance-test.sh"
               }
          }

          stage("Release") {
               steps {
                    sh "kubectl config use-context production"
                    sh "kubectl apply -f hazelcast.yaml"
                    sh "kubectl apply -f deployment.yaml"
                    sh "kubectl apply -f service.yaml"
               }
          }
          stage("Smoke test") {
               steps {
                    sh "kubectl rollout status deployment/calculator-deployment --timeout=180s"
                    sh "chmod +x smoke-test.sh && ./smoke-test.sh"
               }
          }
     }
}