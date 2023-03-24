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
    ''') 
{
  node(POD_LABEL) {
    stage('gradle') {
      container('gradle') {
        git 'https://github.com/jddega/Continuous-Delivery-with-Docker-and-Jenkins-Second-Edition.git'
          stage('Replica Before') {
              sh '''
              curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
              chmod +x ./kubectl
              mv ./kubectl /usr/local/bin/kubectl
              echo 'Number of Replicas before change'
              kubectl get rs -n staging
              kubectl get deployment -n staging
              pwd
              '''
              
          }
          stage('Rolling up') {
             sleep 10
              sh '''
               cd Chapter08/sample1
               echo 'Applying Replicas change'
               kubectl apply -f calculator.yaml -n staging
               kubectl apply -f hazelcast.yaml -n staging
              '''
          }
        
          stage("Checking Replica After") {
             sleep 10
              sh '''
              echo 'Number of Replicas after change'
              kubectl get rs -n staging
              kubectl get deployment -n staging
              '''
        }
      }
    }
  }
}
