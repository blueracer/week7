pipeline {
     agent {
    kubernetes {
      yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          containers:
          - name: gradle
            image: gradle:6.3-jdk14
            command:
            - sleep
            args:
            - 99d
            volumeMounts:
            - name: shared-storage
              mountPath: /mnt        
          - name: kaniko
            image: gcr.io/kaniko-project/executor:debug
            command:
            - sleep
            args:
            - 9999999
            volumeMounts:
            - name: shared-storage
              mountPath: /mnt
            - name: kaniko-secret
              mountPath: /kaniko/.docker
          restartPolicy: Never
          volumes:
          - name: shared-storage
            persistentVolumeClaim:
              claimName: jenkins-pv-claim
          - name: kaniko-secret
            secret:
                secretName: dockercred
                items:
                - key: .dockerconfigjson
                  path: config.json
        '''
    }
  }
  triggers {
          pollSCM('* * * * *')
     }
     stages {
          stage("Compile") {
               steps {
                    sh "./gradlew compileJava"
               }
          }
          stage("Unit test") {
               steps {
                 script {
                   if (env.BRANCH_NAME != 'playground') {
                     sh "./gradlew test"
                   } else {
                     echo "This is a playground branch"
                   }
                 }
               }
          }
          stage("Code coverage") {
               steps {
                 script {
                   if (env.BRANCH_NAME == 'main') {
                    sh "./gradlew jacocoTestReport"
                    sh "./gradlew jacocoTestCoverageVerification"  
                   } else {
                     echo "This is not the main branch"
                   }
                 }
               }
          }
          stage("Static code analysis") {
               steps {
                 script {
                   if (env.BRANCH_NAME != 'playground') {
                    sh "./gradlew checkstyleMain"
                 } else {
                   echo "This is a playground branch"
                 }
                    
               }
          }
        }

          stage("Package") {
               steps {
			       script {
                     try {
					   container('gradle') {
                        sh """
						  ./gradlew build
						  mv ./build/libs/calculator-0.0.1-SNAPSHOT.jar /mnt
						"""
					   }
                     } 
					 catch (Exception e) {
                       echo 'Exception occurred: ' + e.toString()
                       sh 'exit 1'
                      }
                    }
				}
          }

      stage('Build and Push to Docker Registry') {
      steps { 
      container('kaniko') {
        script {
          if (env.BRANCH_NAME != 'playground') {
          stage('Build a container') {
            sh '''
            echo 'FROM openjdk:8-jre' > Dockerfile
            echo 'COPY ./calculator-0.0.1-SNAPSHOT.jar app.jar' >> Dockerfile
            echo 'ENTRYPOINT ["java", "-jar", "app.jar"]' >> Dockerfile
            ls /mnt/*jar
            mv /mnt/calculator-0.0.1-SNAPSHOT.jar .
            '''
            script {
              if (env.BRANCH_NAME == 'main') {
                sh "/kaniko/executor --context `pwd` --destination blueracer/calculator:1.0"
              }
              if (env.BRANCH_NAME == 'feature') {
                sh "/kaniko/executor --context `pwd` --destination blueracer/calculator-feature:0.1"
              }
            }
        }
          } else {
            echo "This is a playground branch"
          }
        }
}
       }
    }
/*
          stage("Docker build") {
               steps {
                    sh "docker build -t leszko/calculator:${BUILD_TIMESTAMP} ."
               }
          }
          stage("Docker login") {
               steps {
                    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'docker-hub-credentials',
                               usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
                         sh "docker login --username $USERNAME --password $PASSWORD"
                    }
               }
          }
          stage("Docker push") {
               steps {
                    sh "docker push leszko/calculator:${BUILD_TIMESTAMP}"
               }
          }
          stage("Update version") {
               steps {
                    sh "sed  -i 's/{{VERSION}}/${BUILD_TIMESTAMP}/g' calculator.yaml"
               }
          }
          
          stage("Deploy to staging") {
               steps {
                    sh "kubectl config use-context staging"
                    sh "kubectl apply -f hazelcast.yaml"
                    sh "kubectl apply -f calculator.yaml"
               }
          }
          stage("Acceptance test") {
               steps {
                    sleep 60
                    sh "chmod +x acceptance-test.sh && ./acceptance-test.sh"
               }
          }
          stage("Release") {
               steps {
                    sh "kubectl config use-context production"
                    sh "kubectl apply -f hazelcast.yaml"
                    sh "kubectl apply -f calculator.yaml"
               }
          }
          stage("Smoke test") {
              steps {
                  sleep 60
                  sh "chmod +x smoke-test.sh && ./smoke-test.sh"
              }
          }
*/
     }
}
