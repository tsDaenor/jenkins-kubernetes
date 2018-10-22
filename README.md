===GOAL===

[OK] minikube
[OK] edit values file
[OK] configuration as code plugin ==> admin paswoord erin, creëer een user (zonder admin rechten, moet geen jobs kunnen maken of editeren, mag ze enkel triggeren)
	--> runnen als job, clone uit git en run op git changes
[OK] jenkins jobbuilder config/container (heeft admin user nodig van hierboven)
[OK] helm install jenkins
[OK] run jenkins job builder to create job
--> run dummy job


===HOWTO===

install virtualbox and minikube to run kubernetes locally
$ brew cask install virtualbox
$ curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.29.0/minikube-darwin-amd64 && chmod +x minikube && sudo cp minikube /usr/local/bin/ && rm minikube
$ minikube version


install command line tool to instruct kubernetes
$ brew install kubectl
$ kubectl version


start minikube and check environment
$ minikube start — vm-driver=virtualbox
$ kubectl get nodes
$ minikube dashboard


install helm (package manager for kubernetes)
$ brew install kubernetes-helm


check namespaces
$ kubectl get ns


Create jenkins namespace and jjb-config configMap in Kubernetes environment
$ mkdir casc
$ mkdir jenkins
$ mkdir -p jenkins-job-builder/jobs
$ vi jenkins/jenkins-namespace.yml
$ vi jenkins-job-builder/jjb-config.yml
$ kubectl create -f jenkins/jenkins-namespace.yml
$ kubectl create -f jenkins-job-builder/jjb-config.yml


install / deploy jenkins (using Helm)
$ helm init
$ vi jenkins/jenkins-values.yml
$ vi casc/config.yaml									# upload to github for using the raw url (in jenkins-values.yml); make sure to define some users in here
$ vi jenkins-job-builder/Jenkinsfile					# upload to github, is used in the first job created in the casc-config
$ vi jenkins-job-builder/pod.yml						# upload to github, is used in the Jenkinsfile
$ vi jenkins-job-builder/jobs/casc.yml 					# upload to github, these jobs will be monitored to be created/updated in jenkins
$ helm install --name jenkins -f jenkins/jenkins-values.yml stable/jenkins --namespace jenkins-project
$ helm ls


deploy changes in configuration
$ kubectl delete pod --namespace jenkins-project jenkins-69b56d945b-bz7vd				# check minikube dashboard to get the name of the jenkins pod


delete and purge helm installation (only in case of issues...)
$ helm delete jenkins
$ helm del --purge jenkins


check services
$ minikube service list
$ minikube service jenkins --namespace jenkins-project
$ kubectl --namespace=jenkins-project exec -ti jenkins-69b56d945b-bz7vd -- bash			# check minikube dashboard to get the name of the jenkins pod
$ kubectl --namespace=jenkins-project logs jenkins-69b56d945b-bz7vd						# check minikube dashboard to get the name of the jenkins pod


stop (and delete!) minikube
$ minikube stop
$ minikube delete
