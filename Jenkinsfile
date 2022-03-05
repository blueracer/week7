pipeline {
     agent any
     triggers {
          pollSCM('* * * * *')
     }
     podTemplate(yaml: '''
    apiVersion: v1
    kind: Pod
    spec:
      containers: 
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
''') 
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

