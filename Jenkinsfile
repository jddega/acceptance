 
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
        git 'https://github.com/jddega/Continuous-Delivery-with-Docker-and-Jenkins-Second-Edition.git'
        stage('Building new feature image') {
          sh '''
            curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
            chmod +x ./kubectl
            mv ./kubectl /usr/local/bin/kubectl
            pwd
            cd Chapter09/sample3
            chmod +x gradlew
            ./gradlew build
          '''
        }
	 stage('Using Kiniko:5 image') {
	   git 'https://github.com/jddega/Continuous-Delivery-with-Docker-and-Jenkins-Second-Edition.git'
	  sleep 10
	  sh '''
	  cd Chapter08/sample1
	  sed -i 's/week8:1.1/hello-kaniko:5.0/g' calculator.yaml
	  kubectl apply -f calculator.yaml -n staging
	  kubectl apply -f hazelcast.yaml -n staging
	  '''
			}
	stage('testing 1') {
	  sleep 60
	  sh '''
	echo 'Addition testing result'
	test $(curl calculator-service.staging.svc.cluster.local:8080/sum?a=3\\&b=4) -eq 7 && echo 'pass' || echo 'fail'

	echo 'Division testing result'
	test $(curl calculator-service.staging.svc.cluster.local:8080/div?a=8\\&b=4) -eq 2 && echo 'pass' || echo 'fail'
	  '''
		}
      stage('Using week8:1.1 image') {
	git 'https://github.com/jddega/Continuous-Delivery-with-Docker-and-Jenkins-Second-Edition.git'
	 sleep 10
	 sh '''
	cd Chapter08/sample1
	sed -i 's/hello-kaniko:5.0/week8:1.1/g' calculator.yaml
	kubectl apply -f calculator.yaml -n staging
	  '''
			}

	stage('Div testing') {
	  sleep 60
	  sh '''
	   echo 'Addition testing result'
	   test $(curl calculator-service.staging.svc.cluster.local:8080/sum?a=3\\&b=4) -eq 7 && echo 'pass' || echo 'fail'

	   echo 'Division testing result'
	   test $(curl calculator-service.staging.svc.cluster.local:8080/div?a=8\\&b=4) -eq 2 && echo 'pass' || echo 'fail'
	  '''
        }
      }
    }
  }
}
