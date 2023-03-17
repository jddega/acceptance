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
          chmod +x gradlew
           ./gradlew build
           cd ..
           cd ..
           pwd
          cd Chapter08/sample1
          pwd
          kubectl apply -f calculator.yaml -n devops-tools
          kubectl apply -f hazelcast.yaml -n devops-tools
          '''
          }
        
        stage("Acceptance test") {
          sleep 60
          sh '''
          kubectl get deployment -n devops-tools
          kubectl get svc -n devops-tools
          kubectl get endpoints -n devops-tools
          test $(curl calculator-service:8080/div?a=6\\&b=2) -eq 3 && echo 'pass' || echo 'fail'
          test $(curl calculator-service:8080/div?a=6\\&b=0) -eq 3 && echo 'pass' || echo 'fail'
          '''
        }
      }
    }
  }
}
