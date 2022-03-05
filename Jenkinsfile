pipeline {
     agent any
     stages {
          stage("Compile") {
               steps {
                    sh "./gradlew compileJava"
               }
          }
          stage("Package") {
               steps {
                    sh "./gradlew build"
                    sh "pwd"
                    sh "ls -l"
               }
          }

     }
}

