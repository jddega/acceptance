podTemplate(yaml: '''
    apiVersion: v1
    kind: Pod
    spec:
      containers:
      - name: gradle
        image: gradle:6.3-jdk14
        command:
        - sleep
        args:
        - 99d
        volumeMounts:
        - name: shared-storage
          mountPath: /mnt
      - name: cloud-sdk
        image: google/cloud-sdk
        command:
        - sleep
        args:
        - 9999999
        volumeMounts:
        - name: shared-storage
          mountPath: /mnt
        - name: google-cloud-key
          mountPath: /var/secrets/google
        env:
        - name: GOOGLE_APPLICATION_CREDENTIALS
          value: /var/secrets/google/molten-crowbar-381403-5fb9f07fe18b.json
      restartPolicy: Never
      volumes:
      - name: shared-storage
        persistentVolumeClaim:
          claimName: jenkins-pv-claim
      - name: google-cloud-key
        secret:
          secretName: sdk-key
    ''') {
  node(POD_LABEL) {
     stage('Testing on gradle') {   
        container('gradle') {
            stage('Installing kubectl') {
               sh '''
              curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
              chmod +x ./kubectl
              mv ./kubectl /usr/local/bin/kubectl
              pwd
              kubectl get pod -n production
              '''
            }
          stage('Build') {
            git 'https://github.com/jddega/Continuous-Delivery-with-Docker-and-Jenkins-Second-Edition.git'
              sh '''
              pwd
              cd Chapter08/sample1
              chmod +x gradlew
              ./gradlew build
              pwd
              '''
            }
           stage('smoke test') {
              sh '''
              cd Chapter08/sample1
              chmod +x gradlew
              ./gradlew smokeTest -Dcalculator.url=http://10.1.3.7:8080
              '''
            }
            
          }
       }
      if (currentBuild.result == 'SUCESS'){
           stage('Deploying to prod') {
                container('cloud-sdk') {
                   stage('Connecting to GKE') {
                      sh '''
                      echo 'namespaces in the staging environment'
                      kubectl get ns
                      gcloud config set project molten-crowbar-381403
                      gcloud auth login --cred-file=$GOOGLE_APPLICATION_CREDENTIALS
                      gcloud container clusters get-credentials hello-cluster --region us-west1 --project molten-crowbar-381403
                      gcloud services enable cloudresourcemanager.googleapis.com pubsub.googleapis.com  container.googleapis.com --project molten-crowbar-381403
                      echo 'namespaces in the prod environment'
                      kubectl get ns
                      gcloud services enable cloudresourcemanager.googleapis.com pubsub.googleapis.com  container.googleapis.com --project molten-crowbar-381403
                      '''
                  }
                   stage('start calculator') {
                      git 'https://github.com/jddega/Continuous-Delivery-with-Docker-and-Jenkins-Second-Edition.git'
                       sh '''
                       cd Chapter08/sample1
                       kubectl apply -f calculator.yaml -n production
                       kubectl apply -f hazelcast.yaml -n production
                       kubectl get pod -n production
                      '''    
               }
                  stage('start calculator') {
                     sleep 60
                     sh '''
                     echo 'I am deployed on google cloud'
                     kubectl get pod -n production
              }     '''   
           }
         }
        
    }
}
