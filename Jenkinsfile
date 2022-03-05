pipeline {
     agent any
     triggers {
          pollSCM('* * * * *')
     }
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

