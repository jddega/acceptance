podTemplate(yaml: '''
    apiVersion: v1
    kind: Pod
    spec:
      containers:
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
   stage('Deploying to prod') {
    container('cloud-sdk') {
      stage('Connecting to GKE') {
        sh '''
        echo 'namespaces in the staging environment'
        kubectl get ns
        gcloud auth login --cred-file=$GOOGLE_APPLICATION_CREDENTIALS
        gcloud container clusters get-credentials hello-cluster --region us-west1 --project molten-crowbar-381403
        echo 'namespaces in the prod environment'
        kubectl get ns
        '''
       }
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
              echo 'Number of Replicas before change'
              kubectl get pod -n staging
              '''
            }
          stage('smoke test') {
            git 'https://github.com/jddega/Continuous-Delivery-with-Docker-and-Jenkins-Second-Edition.git'
              sh '''
              Chapter08/sample1
              chmod +x gradlew
              ./gradlew build
              mv ./build/libs/calculator-0.0.1-SNAPSHOT.jar /mnt
              ./gradlew smokeTest 
              '''
       }
          stage('start calculator') {
            if (currentBuild.result == 'SUCCESS'){
              git 'https://github.com/jddega/Continuous-Delivery-with-Docker-and-Jenkins-Second-Edition.git'
              sh '''
              pwd
              Chapter08/sample1
              kubectl apply -f calculator.yaml -n production
              kubectl apply -f hazelcast.yaml -n production
            '''
        }
        }
       }
      }
     }
    }
   }
  }
 }
}
