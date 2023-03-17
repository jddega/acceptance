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
          cd Chapter09/sample1
          chmod +x gradlew
          ./gradlew build
          kubectl apply -f calculator.yaml -n devops-tools
          kubectl apply -f hazelcast.yaml -n devops-tools
          kubectl get pods -n devops-tools
          kubectl get deployment -n devops-tools
          kubectl get pod
          kubectl get svc -n devops-tools
          kubectl get svc
          kubectl get endpoints -n devops-tools
          kubectl get endpoints
          '''
          }
        
        stage("Acceptance test") {
          sh '''
          cd Chapter09/sample3
          chmod +x gradlew
           ./gradlew build
           ./gradlew acceptanceTest -Dcalculator.url=http://calculator-service:8080
          '''
        }
      }
    }
  }
}
