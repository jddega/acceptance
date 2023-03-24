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
              pwd
              cd Chapter08/sample1
              kubectl apply -f calculator.yaml -n staging
              kubectl apply -f hazelcast.yaml -n staging
              kubectl get pods -n staging
              '''
              
          }
          stage('Rolling up') {
              sleep 10
              sh '''
               cd Chapter08/sample1
               kubectl apply -f calculator.yaml -n staging
               kubectl apply -f hazelcast.yaml -n staging
              '''
          }
        
          stage("Checking Replica After") {
              sleep 10
              sh '''
              kubectl get deployment -n staging
              kubectl get svc -n staging
              kubectl get endpoints -n staging
              kubectl get pods -n staging
              test $(curl calculator-service:8080/div?a=6\\&b=2) -eq 3 && echo 'pass' || echo 'fail'
              test $(curl calculator-service:8080/div?a=6\\&b=0) && echo 'pass' || echo 'fail'
              '''
        }
      }
    }
  }
}
