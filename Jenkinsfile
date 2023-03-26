podTemplate(yaml: '''
    apiVersion: v1
    kind: Pod
    spec:
      containers:
      - name: gradle
        image: gradle:jdk8
        command:
        - sleep
        args:
        - 99d
      restartPolicy: Never
  ''') {
 node(POD_LABEL) {
      stage('gradle') {   
        container('gradle') {
          stage('Actual Replicas') {
              sh '''
              curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
              chmod +x ./kubectl
              mv ./kubectl /usr/local/bin/kubectl
              pwd
              kubectl apply -f calculator.yaml -n staging
              kubectl apply -f hazelcast.yaml -n staging
              echo 'Number of Replicas before change'
              kubectl get deployment -n staging
              kubectl get rs -n staging
              kubectl get pods -n staging
              '''
            }
          stage('Rolling up') {
              git 'https://github.com/jddega/Continuous-Delivery-with-Docker-and-Jenkins-Second-Edition.git'
              sh '''
              cd Chapter08/sample1
              kubectl apply -f calculator.yaml -n staging
              kubectl apply -f hazelcast.yaml -n staging
              '''
          }
          stage("Checking Replicas After") { 
             sleep 60
             sh '''
             echo 'Number of Replicas after change'
             kubectl get deployment -n staging
             kubectl get rs -n staging
             kubectl get pods -n staging
             '''
           }
          }
        }
   }
}
