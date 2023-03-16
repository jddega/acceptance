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
        stage('start calculator') {
          sh '''
          curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
          chmod +x ./kubectl
          mv ./kubectl /usr/local/bin/kubectl
          pwd
          cd Chapter09/sample3
          kubectl apply -f calculator.yaml 
          kubectl apply -f hazelcast.yaml  
          kubectl get pods -n devops-tools
          kubectl get pod
          kubectl get svc
          '''
          }
        
        stage("Acceptance test") {
          sleep 60
          sh '''
          cd Chapter09/sample3
          chmod +x gradlew
          test $(curl calculator-service:30912/sum?a=6\\&b=2) -eq 3 && echo 'pass' || 'fail'
          '''
        }
      }
    }
  }
}
