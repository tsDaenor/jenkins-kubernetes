# Techsession preview: Jenkins as Code

## GOAL

[OK] minikube
[OK] edit values file
[OK] configuration as code plugin ==> admin paswoord erin, creëer een user (zonder admin rechten, moet geen jobs kunnen maken of editeren, mag ze enkel triggeren)
    --> runnen als job, clone uit git en run op git changes
[OK] jenkins jobbuilder config/container (heeft admin user nodig van hierboven)
[OK] helm install jenkins
[OK] run jenkins job builder to create job
    --> run dummy job

## HOWTO

### install virtualbox and minikube to run kubernetes locally

```bash
$ brew cask install virtualbox
$ brew cask install minikube minikube

$ minikube version
minikube version: v0.30.0
```

### install command line tool to instruct kubernetes

```bash
$ brew install kubectl

$ kubectl version
Client Version: version.Info{Major:"1", Minor:"12", GitVersion:"v1.12.0", GitCommit:"0ed33881dc4355495f623c6f22e7dd0b7632b7c0", GitTreeState:"clean", BuildDate:"2018-09-28T15:20:58Z", GoVersion:"go1.11", Compiler:"gc", Platform:"darwin/amd64"}
Server Version: version.Info{Major:"1", Minor:"10", GitVersion:"v1.10.0", GitCommit:"fc32d2f3698e36b93322a3465f63a14e9f0eaead", GitTreeState:"clean", BuildDate:"2018-03-26T16:44:10Z", GoVersion:"go1.9.3", Compiler:"gc", Platform:"linux/amd64"}
```

### start minikube and check environment

```bash
$ minikube start — vm-driver=virtualbox
Starting local Kubernetes v1.10.0 cluster...
Starting VM...
Getting VM IP address...
Moving files into cluster...
Setting up certs...
Connecting to cluster...
Setting up kubeconfig...
Starting cluster components...
Kubectl is now configured to use the cluster.
Loading cached images from config file.

$ kubectl get nodes
NAME       STATUS   ROLES    AGE   VERSION
minikube   Ready    master   17m   v1.10.0

$ minikube dashboard
Opening kubernetes dashboard in default browser...
```

### install helm (package manager for kubernetes)

```bash
$ brew install kubernetes-helm
```

### check namespaces

```bash
$ kubectl get ns
```

### Create jenkins namespace and jjb-config configMap in Kubernetes environment

```bash
$ mkdir casc
$ mkdir jenkins
$ mkdir -p jenkins-job-builder/jobs
$ vi jenkins/jenkins-namespace.yml
$ vi jenkins-job-builder/jjb-config.yml

$ kubectl create -f jenkins/jenkins-namespace.yml
namespace/jenkins-project created

$ kubectl create -f jenkins-job-builder/jjb-config.yml
configmap/jjb-config created
```

### install / deploy jenkins (using Helm)

```bash
$ helm init
Creating /Users/foob/.helm
Creating /Users/foob/.helm/repository
Creating /Users/foob/.helm/repository/cache
Creating /Users/foob/.helm/repository/local
Creating /Users/foob/.helm/plugins
Creating /Users/foob/.helm/starters
Creating /Users/foob/.helm/cache/archive
Creating /Users/foob/.helm/repository/repositories.yaml
Adding stable repo with URL: https://kubernetes-charts.storage.googleapis.com
Adding local repo with URL: http://127.0.0.1:8879/charts
$HELM_HOME has been configured at /Users/foob/.helm.

Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.

Please note: by default, Tiller is deployed with an insecure 'allow unauthenticated users' policy.
To prevent this, run `helm init` with the --tiller-tls-verify flag.
For more information on securing your installation see: https://docs.helm.sh/using_helm/#securing-your-helm-installation
Happy Helming!
```

```bash
$ vi jenkins/jenkins-values.yml
$ vi casc/config.yaml
# upload to github for using the raw url (in jenkins-values.yml); make sure to define some users in here
$ vi jenkins-job-builder/Jenkinsfile
# upload to github, is used in the first job created in the casc-config
$ vi jenkins-job-builder/pod.yml
# upload to github, is used in the Jenkinsfile
$ vi jenkins-job-builder/jobs/casc.yml
# upload to github, these jobs will be monitored to be created/updated in jenkins
```

```bash
$ helm install --name jenkins -f jenkins/jenkins-values.yml stable/jenkins --namespace jenkins-project
NAME:   jenkins
LAST DEPLOYED: Tue Oct 23 08:33:49 2018
NAMESPACE: jenkins-project
STATUS: DEPLOYED

RESOURCES:
==> v1/Secret
NAME     AGE
jenkins  0s

==> v1/ConfigMap
jenkins        0s
jenkins-tests  0s

==> v1/ServiceAccount
jenkins  0s

==> v1/ClusterRoleBinding
jenkins-role-binding  0s

==> v1/Service
jenkins-agent  0s
jenkins        0s

==> v1/Deployment
jenkins  0s

==> v1/Pod(related)

NAME                      READY  STATUS    RESTARTS  AGE
jenkins-846b4b4cc6-lwq2x  0/1    Init:0/1  0         0s


NOTES:
1. Get your 'admin' user password by running:
  printf $(kubectl get secret --namespace jenkins-project jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo
2. Get the Jenkins URL to visit by running these commands in the same shell:
  export NODE_PORT=$(kubectl get --namespace jenkins-project -o jsonpath="{.spec.ports[0].nodePort}" services jenkins)
  export NODE_IP=$(kubectl get nodes --namespace jenkins-project -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT/login

3. Login with the password from step 1 and the username: admin

For more information on running Jenkins on Kubernetes, visit:
https://cloud.google.com/solutions/jenkins-on-container-engine
#################################################################################
######   WARNING: Persistence is disabled!!! You will lose your data when   #####
######            the Jenkins pod is terminated.                            #####
#################################################################################
Configure the Kubernetes plugin in Jenkins to use the following Service Account name jenkins using the following steps:
  Create a Jenkins credential of type Kubernetes service account with service account name jenkins
  Under configure Jenkins -- Update the credentials config in the cloud section to use the service account credential you created in the step above.
```

```bash
$ helm ls
NAME   	REVISION	UPDATED                 	STATUS  	CHART         	APP VERSION	NAMESPACE
jenkins	1       	Tue Oct 23 08:33:49 2018	DEPLOYED	jenkins-0.19.1	2.121.3    	jenkins-project
```

### deploy changes in configuration

```bash
# check minikube dashboard to get the name of the jenkins pod
$ kubectl delete pod --namespace jenkins-project jenkins-846b4b4cc6-lwq2x
pod "jenkins-846b4b4cc6-lwq2x" deleted
```

### delete and purge helm installation (only in case of issues...)

```bash
$ helm delete jenkins
release "jenkins" deleted

$ helm del --purge jenkins
release "jenkins" deleted
```

### check services

```bash
$ minikube service list
|-----------------|----------------------|-----------------------------|
|    NAMESPACE    |         NAME         |             URL             |
|-----------------|----------------------|-----------------------------|
| default         | kubernetes           | No node port                |
| jenkins-project | jenkins              | http://192.168.99.100:32000 |
| jenkins-project | jenkins-agent        | No node port                |
| kube-system     | kube-dns             | No node port                |
| kube-system     | kubernetes-dashboard | http://192.168.99.100:30000 |
| kube-system     | tiller-deploy        | No node port                |
|-----------------|----------------------|-----------------------------|

$ minikube service jenkins --namespace jenkins-project
Opening kubernetes service jenkins-project/jenkins in default browser...

$ kubectl --namespace=jenkins-project exec -ti jenkins-69b56d945b-bz7vd -- bash
# check minikube dashboard to get the name of the jenkins pod
$ kubectl --namespace=jenkins-project logs jenkins-69b56d945b-bz7vd
# check minikube dashboard to get the name of the jenkins pod
```

### stop (and delete!) minikube

```bash
$ minikube stop
Stopping local Kubernetes cluster...
Machine stopped.

$ minikube delete
Deleting local Kubernetes cluster...
Machine deleted.
```
