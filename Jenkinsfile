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
        git 'https://github.com/jddega/Continuous-Delivery-with-Docker-and-Jenkins-Second-Edition.git'
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
      }
     }
   }
}
