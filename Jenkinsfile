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
        git
      'https://github.com/jddega/Continuous-Delivery-with-Docker-and-Jenkins-Second-Edition.git'
        stage('start calculator') {
          sh '''
          cd Chapter09/sample1
          curl -LO "https://dl.k8s.io/release/$(curl -L -s
          https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl)"
          chmod +x ./kubectl
          mv ./kubectl /usr/local/bin/kubectl
          ./kubectl apply -f calculator.yaml -n devops-tools
          ./kubectl apply -f hazelcast.yaml -n devops-tools
          '''
          }
        stage("Acceptance test") {
     steps {
          sleep 60
          sh "test $(curl calculator-service:8080/sum?a=6\\&b=2) -eq 3 && echo 'pass' || 'fail"
          }
        }
      }
    }
 }
