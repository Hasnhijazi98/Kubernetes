*************************************************************************
************************KUBERNETES FOR BEGINNERS*************************
*************************************************************************

First, we'll install some small things to run things at our pc, install Minikube :

for that install kubectl first : 
-sudo snap install kubectl --classic  (i just love to use snapshot versions for simpler installation)
-enable virtualization, in my case, on a ubuntu, i searched for the vmx virtualization command grep -E --color 'vmx|svm' /proc/cpuinfo to see if virtualization is already on
-then install minikube choosing the right settings for you from this link : https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fbinary+download
-you may need in between the steps to disable the secure boot or put a MOK key, it's simple and easy, but in the choice of mok key, the keyboard keys during the password logging at the reboot is in english-US format

For windows :
-install the kubernetes from the official kubernetes website, make choose the suitable version and architecture for your pc (you can use curl or chocolatey...)
-then install minikube
-make sure virtualization is enabled, by entering bios menu and enabling intel-vt or amd-v according to ur system

then you can start minikube and choose the driver you want to run on: minikube start --driver=virtualbox (you can choose another driver like docker,qemu2...)
NOTE: on windows, in case of having error at creating a vm using minikube for virtualbox, you can either use other vm like hyper-v, or use docker, but else then add the command
 minikube start --driver=virtualbox --no-vtx-check

now let's begin, to create a pod, launch this cmd : kubectl run [name] --[configurations] - the configurations must have at least a image (--image=[image])

**************CREATING PODS YAML:********************************

always when doing some config for pods, replica or deployment, the yaml file must contain these fields

	apiVersion:
	kind: Pod/replica...
	metadata:
	  name:
	  labels:
	    app:
	spec:
	
in my example, i created a pod using this :

	apiVersion: v1
	kind: Pod
	metadata:
	  name: nginx
	  labels:
	    app: nginx
	    tier: frontend
	spec:
	  containers:
	  - name: ningx-tut
	    image: nginx

to add environment variables, add it under the containers, sibling to name and image and it's a collection, example :
 
  containers:
    - name: postgres
      image: postgres
      env:
        - name: POSTGRES_PASSWORD
          value: mysecretpassword

and then you can check it in the pods list : kubectl get pods / and if you want more info about ur pod : kubectl description [pod name in metadata]

so in general, when creating a pod using kubectl commands, the pod is scheduled to run on one node of the nodes in the cluster, and it's the scheduler that decides on which one

**************CREATING REPLICASET YAML:********************************


Now to deploy multiple pods on one or more nodes, we can use for that the replica-controllers, or the replica-set, both conf are written in yaml files, and there are really similar
as the replica set have a additional part named selector

the replicaset def file have the first part common with the conf file of pods, the part of ApiVersion,kind and metadata then we define in the spec a selector:
in the selector, we'll have a matchLabel, so the  replicaset can manage the pods that already have the same value for the field described in the matchLabel, the replicas number
and a template that will have the ordinary definition of the pod, example :

	apiVersion: apps/v1
	kind: ReplicaSet
	metadata:
	  name: replicaset
	  labels:
		app: replicaset
	spec:
	  selector:
		matchLabels:
		  app: replicaset-app
	  replicas: 4
	  template:
		  metadata:
			name: nginx
			labels:
			  app: replicaset-app
		  spec:
			containers:
			  - name: nginx
				image: nginx

let's break it down shall we ?

firs part, like the pod def file
seconde part, in spec, we design a selector and replicas number of the pods we'll create/manage
3rd part , under spec.template, we'll define the conf like a normal pod app

to give the replicaset management a shot, use kubectl get pods command to see the pods created,in our example we'll see 4, try to delete one and re-run the get pods cmd, you'll
see the number of pods still the same but a new one replaced the deleted one, same thing if you try to scale up/down the replicas number, pods will be terminated or will be created


**************CREATING DEPLOYMENT FILE:********************************

well, the code is the exact same thing as the replicaset conf yaml file, but with "Deployment" as a value for the field kind instead of "ReplicaSet"
the deployment is the bigger unit of replicaset ; so it contains the replicaset, and the replicaset contains the pod(s)
so the logic behind the deployment of kubernetes is that the deployment contains one replica-set, when a new version is deployed, it will replace the existing pods one after the other
so we won't have some unavailability of the apps, that's called the rollout method, then the new replica-set is created and the old one is kept aside and tracked, so when something
goes wrong, we can just do a rollout command and the app returns to its older version, to do so:

	kubectl rollout undo deployment/<deployment_name>
	
and also watch out, for the deployment type, you should precise the strategy of the deployment, it is set as a sibling for replicas and selector under the spec


****************SERVICES**************************************************

when we, on a pc, want to connect to a pod, it is essentail to pass by services, they provide the connection between what's inside a Node and our end user, as the pods are 
being on an internal network inside the slave node and their ip is assigned to it (and also provides load-balancing, resiliency...)
let's take a use case : on my pc, i want to connect to pod(s) and make some commands, to connect, i must create a service that will manage the connection inside the node(s) so
that the pods inside will be exposed to a certain port, this port must be between 30000 and 32767, the service is defined by a yaml file (oh what a surprise) and is similar
to a replicaset, just with a part where we define the type (there is many service types), the Node port, the target port of the pod and the service port to connect to the pod(s),
and also we add a selector, so the selected pool of  pods is according to labels we add to them
and at last you can search for the ip adresse of the node, copy it in browser and then the node port and you'll be good to go !

example of a service of type "NodePort" :

	apiVersion: v1
	kind: Service
	metadata:
	  name: service
	spec:
	  type: NodePort
	  ports:
		- port: 80
		  targetPort: 80
		  nodePort: 30004
	  selector:
		app: deployment-app

i think it's pretty clear what we've done here, no need to break it down
WATCH OUT THE SELECTOR HERE DOES NOT HAVE A 'matchLabels' FIELD !!!
the other services in kubernetes are Service and ClusterIp

Remeber in the docker course how we deployed microservices app with docker ? yeah yeah the voting-app

we ran instances of images and linked eahc one to another using the --link: redis=redis something like that
the application we'll use in the following is voting-app example (kodekloud/examplevotingapp)
now to run a full microservices application using kubernetes, we must create a pod for each of the 5 microservices, and then services for the pods that are linked with other pods

WATCH OUT: when pulling an image in the field - spec.containers.image, be sure to put the right tag
you can see the code for the pods and services of the microservices in the application

deploying in clouds:

For GCP, AWS (EKS) and Azure (AKS), the process is pretty similar, create/add a new kubernetes cluster, it'll demand you how many nodes you want, and then after some minutes
you can access the cloud command line to clone the project and code to the nodes, and then create -f <deployment/service_files> one after the other, the type of the service here
in the front apps will be changed from "NodePort" to "LoadBalancer"

after all is deployed succesfully, you will be able to access your app from anywhere using the adress provided by the cloud server
(In aws, you must create first a IAM and asses to it the kubernetes clusters creation permissions)

dictionnary commands:

to get more info about nodes/pods				kubectl get nodes/pods -o wide
to get more info about specific pod				kubectl describe pod <pod_name> 
to delete a specific pod						kubectl delete pod <pod_name>
to edit a pod configurations yml file			kubectl edit pod <pod_name>
to create/update a pod from a file 				kubectl apply -f <file_name>.yaml / kubectl create -f <file_name>.yaml
to create a replicaset/controllers/deployment	kubectl create -f <file_name>.yaml
to delete a replicaset(deletes under-pods) 		kubectl delete replicaset <replicaset_name>
to update a replicaset file						kubectl replace -f <file_name>.yaml
to scale up/down a replicaset					kubectl scale --replicas=<number> -f <file_name>.yaml / kubectl scale --replicas=<number> <replica-set_name>
to edit a replicaset (edits made in-memory)		kubectl edit replicaset <replicaset_name>
to get all info (pods+replicas+deployments) 	kubectl get all
to get the rollout history/status				kubectl rollout history/status <deployment_name>
to record the rollout and changes				kubectl create -f <file_name> --record=true
to rollback from a deployment					kubectl rollout undo deployment/<deployment_name>
