# Kubernetes Intro:

    Google introduced this in 2014 to scale docker containers and auto heal them.. In other word to Orcestrate the docker containers..

    Earlier services used to run as monolith (all in one place). But problem was if one service goes down, the whole thing used to go down..
        So Microservices concept was introduced....

        PODS:
          A Pod is the smallest deployable unit in Kubernetes
            It can have:
            1 container (most common) ✅
            OR multiple tightly coupled containers (sidecar pattern)

        Cluster:
          A Cluster contains Nodes
            Nodes run Pods

# BASIC Kunernetes Structure:

    there are mainly 2 kinds of server/node that constructs Kubernetes (K8S).
        - Master Node (Control Plane) : This functions to mainly get the work done by the worker node. It has:
                  API Server
                  Scheduler
                  Controller Manager
                  etcd

                    API Server receives requests (via kubectl)
                    Scheduler decides which node
                    Kubelet runs pods on that node

        - Worker Node: This mainly runs and manages all the docker containers.
                  Kubelet
                  Container Runtime (Docker / containerd)
                  Kube-proxy


    - Kubectl: Kubectl is CLI tool used by us (client) to talk to API Server


            ##MASTER NODE       |--------- |             ##Worker Node 1
            --------------------|----      |         ----------------
           | Scheduler          |   |   ##Kubectl    | Kublet       |
           |                    |   |                |              |  ##USER
           | etcd        API SERVER |                | Pods         |    |
           |                        |                |              |    |
           |Controller              |                | Service Proxy|    |
           | Manager                |                 ---------|-----    |
           |________________________|                          |_________|
             ___________________________________________________________
            |                                                            |
            | ### CNI network: Weave net, Calico etc.                        |
            |____________________________________________________________|
                    CNI means Container Network Interface

# Kubernetes Clusters:

    -Kubeadm: eg.. take 3 different ec2 instances and cluster them. But it proves costly..

    -Minikube: can be created in local or a single ec2 instance. It is a single node cluster.

    -KIND Cluster: It stands for Kubernetes in Docker..

    -EKS, AKS, GKE: mainly on cloud (Elastic Kubertnetes Serevices (AWS), Azure Kubernetes Service (Azure), Google Kubernetes Engine (Google))

    -There are other companies as well who provides kubernetes engines like RKE (Rancher Kubernetes),  CIVO etc..

# KIND Cluster:

      #Step 1:
        Create an ec2 instance in AWS.. (t2 large)

        make sure docker installed.. or install docker

        install kind and kubectl with a bash script.. write the script.... you may get the script from github as well... (https://github.com/LondheShubham153/kubestarter) ... try to go though and learn how the script is written...   run the script to install kind and kubectl

        then write a .yml (config.yml) file for configuration in the repo where you want to create the cluster

         example: we are now configuring for creating 1 master node and 3 worker node..

```YAML
            kind: Cluster
            apiVersion: (you can get this in the kind website) (kind.x-k8s.io/v1alpha4)

            nodes:
            - role: control-plane (this means master node)
              image: kindest/node:v(lastest version) (kindest/node:v1.29.0)
            - role: worker
              image: kindest/node:v(lastest version)
            - role: worker
              image: kindest/node:v(lastest version)
            - role: worker
              image: kindest/node:v(lastest version)
              extraPortMappings:
              - containerPort: 80
                hostPort: 80
                protocol: TCP
              - containerPort: 443
                hostPort: 443
                protocol: TCP
```

#Step 2:
inside the folder where you would like to create this cluster:

```
kind create cluster --name (name the cluster) --config=config.yml
```

to check:

```
kubectl cluster-info --context kind-(cluster name)
```

to ckeck the nodes in the cluster:

```
kubectl get nodes --context kind-(cluster name)
```

to set deafult a particular cluster:

```
kubectl config use-context kind-(cluster name)
```

# HOW KUBERNATES WORK:

    KUBERNETES WORKS IN A GROUPED WAY CALLED NAMESPACE.

      Recources of a Namespace is :

      we need to run a docker container :
                DOCKER
                   |
                   |
       Many containers form a pod
                  POD
                   |
                   |
      For autoscaling and autohealing we need
                DEPLOYMENT
                   |
                   |
      For provioding access to the outside world
                  SERVICE

# NAMESPACE :

    TO get the namespaces:

```
      kubectl get namespace
            or
      kubectl get ns
```

    the namespace is very important, as there may be pods running in certain name sapces and may be no pods running at all... so to check:
      kubectl get pods -n (name of namespace...)

    # create a ns:

```
      kubectl create ns (name)
        eg.. kubectl create ns nginx
```

    Default namespaces exist:
        default
        kube-system
        kube-public
        kube-node-lease

# PODS:

    to create a pod:

```
      kubectl run (pod_name) --image=(DOCKER_image_name) -n (ns_name)
        eg: kubectl run nginx --image=ngnix -n nginx
```

    to check a pod:

```
      kubectl get pods -n (ns_name)
        eg.. kubectl get pods -n nginx
```

# Create everything through a MANIFEST file:

    till now we were using command lines... now we will learn about all these recources through a .yml (Manifest) file...

    # creating ns through manifest file
        -> vim namespace.yml
        ->

```YAML
            kind: Namespace
            apiVersion: v1
            metadata:
              name: nginx
```

->

        ->

```
        kubectl apply -f namespace.yml
```

    # creating pods through manifest file:
        -> vim pod.yml
        ->

```yaml
kind: Pod
apiVersion: v1
metadata:
  name: nginx-pod
  namespace: nginx
spec:
  containers:
    - name: nginx
      image: nginx:1.14.2
      ports:
        - containerPort: 80
```

        ->

```
        kubectl apply -f pod.yml
```

        -> to get inside any pod with interactive terminal:

```
        kubectl exec -it nginx-pod -n nginx -- bash
```

        -> to get detailed reports of pods:

```
        kubectl describe pod/nginx-pod -n nginx
```

# DEPLOY:

.. This is the process of making the pods scalable. Deployment creates the pods in replica sets instead of creating just 1 pod at a time....

This is achived by creating replicas of the pods that we have made, so that when a lot of user interacts suddently, then we can easily scale the pods as per our need.

This process is controlled by REPLICATION CONTROLLER.

There are 6 kinds of workloads:

1. ReplicaSet
2. StatefulSet
3. Deployment
4. DaemonSet
5. Job
6. CronJob

All of the above are used in the same process of creating replicas of the pods, its just that each of the set maintains the replica differently...

-> ReplicaSet just creates the number of duplicates that the engineer wants

-> StatefulSet, after creating the replicas, this creates its state (through numbering the replicas)

-> Deployment is very similar to ReplicaSet, but it gives an option of ROLLING UPDATES to the pods.
.) This means that in this way, when the eng pushes updates, all the replicas does not start updating at once... A few pod-replicas starts updating, while the other ones are maintained at the previous state. Only when the initial sets are done, then the other ones will get updated..

-> DaemonSet: in other sets, the no. of worker node is not managed equally. In this kind of workload, it ensures that each worker node gets a replica...
if there are 3 worker nodes, DaemonSet will ensure that each worker node should get atleast 1 replica...
we do not have to specify the replicas no. in DaemonSet, it will automatically create the no. of replicas as many as the worker nodes

-> Jobs: these are one-off tasks that is inside a container... we want it to run and get over with...
eg... taking a back up, updating, patching etc..

-> CRON Jobs: these are tasks that run on a schedule created by us or the system..
for this we should know about CRON (schedule)
_(min 0-59) _(hour 0-23) _(day of month 1-31) _(month 1-12) \*(day of week 0-6 or names)
A CronJob object uses one cron-format schedule string in .spec.schedule. Kubernetes describes a CronJob as one line of a Unix crontab and uses cron format in the schedule field.

Explanation:
Think of cron as a calendar rule.
eg..

1.  0 9 \* \* _ (Meaning of _: every possible value for that
    field)
    means: you are saying:
    minute = 0
    hour = 9
    day of month = every day
    month = every month
    day of week = every day of week
    in other words: Run every day at 9:00 AM

2.  - - - - -     means: every minute
3.  0 0 \* \* \* means: Every day at midnight
4.  30 18 \* \* \* means: Every day at 6:30 PM
5.  0 2 \* \* 0 means: Every Sunday at 2:00 AM
6.  15 1 1 \* \* means: On the 1st day of every month at
    1:15 AM
    Read left to right:

EXPLANATION OF EACH OF THE ABOVE BY EXAPMLES:

For the REPLICATION CONTROLLER to identify the pod that will be created, we must give the following in the manifest file when creating deployemnt:
-> LABEL : we must provide a label to the pod
-> SELECTOR: we must provide a selector to the deployment to select the labeled pod

the logic is, when the label name and selector name match, the pod is replicated..

## creating deployment of the pods through manifest file:

just like a manifest file of pods, its the same, but here
the METADATA should define the LABELS of the deployemnt
and
the SPEC (SPECIFICATION) file of the deployment should give:
.) as decribed above, we must provide REPLICAS details (no. of replica required)....

.) a SELECTOR to match with the label (MATCHLABELS)...

.) also define the pods in a TEMPLATE... this template should define 2 key values:

    - METADATA of the Pods to be created: which should define:
        LABELS of the pod
    - SPEC of the pods to be created

Examples with explanations:

## 1. DEPLOYMENT:

-> vim deploment.yml
->

```yaml
kind: Deployment
apiVersion: apps/v1
### METADATA OF DEPLOYMENT
metadata:
  name: nginx-deployment
  namespace: nginx
  labels:
    app: nginx
### SPECIFICATION OF DEPLOYMENT
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx
  ### TEMPLATE FOR POD
  template:
    ### METADATA OF POD
    metadata:
      labels:
        app: nginx
    ### SPEC OF POD
    spec:
      containers:
        - name: nginx
          image: nginx:1.14.2
          ports:
            - containerPort: 80
```

-> check:

```
kubectl get pods -n nginx
```

and you will see 4 pods ready....

### HOW DOES IT HELP TO SCALE UP?

in times of high traffic.. you can simply do:

```
kubectl scale deployemnt/nginx-deployment -n ngnix --replicas=10
```

you will see 10 pods ready straight away...

###NOTE: at any point of time if you want a descriptive pod details:

```
kubectl get pods -n nginx -o wide
```

this will show you which worker node is scheduled for which pod etc..

### HOW DOES DEPLOYMENT HELP WITH ROLL OUT UPDATES:

Lets say we were working with nginx version 1.26.2 and now we want to update it to nginx version 1.27.3... in such a case we will run:

kubectl set image deployment/nginx-deployment -n nginx ngnix=nginx:1.27.3

as soon as we do this, we will see that few pods are rolling updates, wheras others are still on the previous vesrions...
only when the previous updating pods are fully updated, they will be up, and then the rest will get updated...

this way, the service will never be down...

## 2. REPLICASET:

-> vim replicaset.yml
->

```YAML
kind: ReplicaSet
apiVersion: apps/v1
### METADATA OF DEPLOYMENT
metadata:
  name: nginx-replica
  namespace: nginx
  labels:
    app: nginx
### SPECIFICATION OF DEPLOYMENT
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx
  ### TEMPLATE FOR POD
  template:
    ### METADATA OF POD
    metadata:
      labels:
        app: nginx
    ### SPEC OF POD
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

-> check:

```
kubectl get pods -n nginx
```

and you will see 4 pods ready....

## 3.DAEMONSET:

-> vim daemonset.yml
->

```YAML
kind: DaemonSet
apiVersion: apps/v1
### METADATA OD DEPLOYMENT
metadata:
  name: nginx-daemon
  namespace: nginx
  labels:
    app: nginx
### SPECIFICATION OF DEPLOYMENT
spec:
  ### note: no replica specified here
  selector:
    matchLabels:
      app: nginx
  ### TEMPLATE FOR POD
  template:
    ### METADATA OF POD
    metadata:
      labels:
        app: nginx
    ### SPEC OF POD
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```

-> esc :wq enter

-> check:

```
kubectl get pods -n nginx
```

and you will see 3 pods ready... as there are 3 worker nodes...

## 4. JOBS:

This is different from the other sets, as it is a job that we want to run with our instructions...

here in SPEC, we need parameters like:
COMPLETIONS: how many successful runs you want.
PARALLELISM: how many Pods may run at the same time.
TEMPLATE:
METADATA: this follows same structure as other sets above.. change the names accordingly..
SPEC:
-name: batch-container
image: busybox:latest **\*\***\*\*\*\***\*\***
(this is a container whose function is just to run command.... just like nginx function is to serve the web pages)
command: ["sh", "-c", "echo hello world && sleep 10"]
(this means run a command in shell and then sleep after 10 seconds)
RESTART POLICY: Never
(if the job fails or the work is done, would you like to restart this?)

Example:
-> vim jobs.yml
->

```yaml
kind: Job
apiVersion: batch/v1
metadata:
  name: demo-job
  namespace: nginx
spec:
  completions: 1
  parallelism: 1
  template:
    metadata:
      labels:
        apps: batch-job
    spec:
      containers:
      - name: batch-container
        image: busybox:latest
        command: ["sh", "-c", "echo hello world && sleep 10]
    restartPolicy: Never
```

-> esc :wq enter

->

```
kubectl apply -f job.yml
```

-> check:

```
kubectl get jobs -n nginx
```

you will see the pod state as running for first 10s and then change the state to complete after 10s

- from here we also learn that pods have differnet states eg. container creating, running, terminating, completed etc..

-> check logs:

```
kubectl logs pod/demo-job-qqrh(pod_name) -n nginx
```

you will see: hello world..

## 5. CRONJOBS:

A CronJob is not the Pod itself.
The actual chain is: CronJob → Job → Pod

CronJob decides when
Job decides run this task now
Pod actually runs the container

very important partS of CRONJOB is the SCHEDULE (when to run the job).. and the JOB-TEMPLATE that defines the job.

Basic CronJob YAML structure:

```YAML
apiVersion: batch/v1
kind: CronJob
metadata:
  name: demo-cronjob
  namespace: nginx
spec:
  schedule: "*/1 * * * *"
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: batch-container
            image: busybox:1.36
            command:
            - /bin/sh
            - -c
            - date; echo "Hello from CronJobs"
        restartPolicy: OnFailure
```

# STORAGE:

## Definations:

Persistent Volumes (PV): A volume is storage that Kubernetes makes available to the Pod.
Volume Mount: A volume mount is the place where that storage box appears inside the container.
Persistent Volume Claim (PVC): A Pod does not directly ask for a disk in most beginner setups. Instead it says: I need storage of this size. That request is the PVC. Then Kubernetes connects that PVC to actual storage.

## PERSISTENT VOLUME (PV):

the whole idea is to attach (persist) the data inside the pod with the host machine (PV = storage resource in Kubernetes cluster (it may be local disk, EBS, NFS, cloud storage etc.)) so that when a pod is lost, then the new pod created should get this data from the host machine/EBS/CLOUD. this storage inside the host machine/others.. is PV.
We gebrally allocate such volumes manually. lets say if our host machine is 30gb, so we say to kubernetes, pesist a volume of 1Gi for my pod data.

WHAT IS THE NEED OF VOLUMES:
Inside Kubernetes, a Pod is temporary. So, when the Pod dies, the files inside that container also disappear.
Don't Confuse: Kubernates deployment will definetely create another pod as per replicas set earler, as soon as a pod die, but the data inside that pod is not bound to any volume yet. So the data inside that pod gets lost.

Example yml of PV:

```YAML
kind: PersistentVolume
apiVersion: v1
metadata:
  name: local-pv
  namespace: nginx
  labels:
    app: local
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain # This will help in retaining this volume
  storageClassName: local-storage #this clould be cloud-storage, network-storage
  hostPath:
    path: /mnt/data
```

now after applying.. when you do

```
kubectl get pv
```

you will see, 1Gi is available

PERSISTENT VOLUME CLAIM (PVC):

Now after creating the PV, we need to claim it for our use. So we need PVC.

Example PVC yml:

```YAML
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: local-pvc
  namespace: nginx
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: local-storage
```

~
now after applying, when you do:

```
kubectl get pv
```

you will see that there is a 1Gi pv, but the status will say bound...~

VERY IMPORTANT — PV ↔ PVC MATCHING:
You must understand this clearly: PVC requests → PV provides
Matching happens based on::

```
              | Field            | Must match |
              | ---------------- | ------------------ |
              | storageClassName | ✅ |
              | size             | PVC ≤ PV |
              | accessModes      | must be compatible |
```

Example from previous case:
PV:
storage: 1Gi
storageClassName: local-storage
PVC:
storage: 1Gi
storageClassName: local-storage
👉 So Kubernetes binds them

IMPORTANT REALITY YOU MUST KNOW:
👉 This study notes are using: hostPath - ONLY for learning ❗
In real production:
You use:
AWS EBS
Azure Disk
GCP Persistent Disk
NFS
Cep

🧠 FINAL MEMORY MAP (VERY IMPORTANT)
Pod → uses volumeMount
volumeMount → connects to volume
volume → connects to PVC
PVC → claims PV
PV → actual storage

NOW YOU CAN APPLY THIS IN DEPLOYMENT OR REPLICASET:

make sure you delete the previous deployemnt... then edit the deployment.yml ->

```YAML
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nginx-deployment
  namespace: nginx
  labels:
    app: nginx
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
        volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: my-volume
      volumes:
        - name: my-volume
          persistentVolumeClaim:
            claimName: local-pvc
```

# SERVICE:

After the pods are deployed, this needs to get accessible in the outside world by a user. This is done via the Services..
In other words, Services are used to expose the pods..

A Service exposes Pods over the network. Depending on the Service type, it may expose them:
-> only inside the cluster
-> through node ports
-> through an external load balancer

WHY SERVICE IS NEEDED:

Suppose you have a Deployment with 4 nginx Pods.
Those Pods have:
->their own Pod IPs
->temporary lifecycle
->random names
->they can be recreated anytime
So if one Pod dies, its IP can disappear and a new Pod may come with a different IP.
That means you should not connect users directly to Pod IPs.
So Kubernetes gives you a Service.

A Service is a stable network front door for a set of Pods selected by labels. Kubernetes assigns a Service a stable virtual IP (clusterIP) for its lifetime, and traffic sent to that Service is load-balanced to matching Pods.

Deployment creates Pods -> Service finds those Pods using labels -> Clients talk to Service -> Service forwards traffic to one of the Pods

Example of yml file of Services:

```YAML
kind: Service
apiVersion: v1
metadata:
  name: nginx-service
  namespace: nginx
spec:
  selector:
    app: ngnix   # this is done to select the app for exposing
  ports:
    - protocol: TCP
      port: 80  #(This is the port on the Service itself)
      targetPort: 80 #(This is the port on the Pod/container)
  type: ClusterIP #(other values: NodePort, Loadbalancer etc., Headless) *** FOR DWESCRIPTION SEE BELOW
```

Now after apply, if you want to cheack ALL (pods, deploment, service):

```
kubectl get all -n ngnix
```

Problem: this cluster is in a docker container..so we need to forward the port to access pods from outside:

```
sudo -E kubectl port-forward service/ngnix-service -n nginx 80:80 --address=0.0.0.0
```

now make sure that you expose the port 80 on ec2 instance

But remember:
kubectl port-forward is mainly for testing/debugging
it is not the normal production exposure method

## SERVICE TYPES:

ClusterIP, NodePort, LoadBalancer, and ExternalName
(A headless Service is created by setting clusterIP: None; it is a Service pattern rather than a separate type value.)

### ClusterIP — step by step:

EXAMPLE:

```YAML
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  namespace: nginx
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

What it does?
It gives the Service an internal virtual IP inside the cluster.
That IP is reachable from: other Pods, cluster-internal clients
But not directly from the public internet.ClusterIP is cluster-internal; Kubernetes allocates a virtual IP and routes traffic from that Service to backing Pods.
When to use it?
Use ClusterIP when:
-> one app inside the cluster needs to talk to another app
-> frontend Pod talks to backend Pod
-> backend Pod talks to database Service
-> you only need internal communication
Why use it?
Because it gives:
-> stable internal address
-> load balancing across Pods
-> no need to track Pod IPs
Example situation:
Suppose you deploy: frontend & backend
The frontend can call: http://backend-service:5000 inside the cluster.

### NodePort — step by step:

EXAMPLE:

```YAML
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
  namespace: nginx
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
    nodePort: 30080
  type: NodePort
```

What it Does:
Kubernetes opens a port on every node in the cluster, usually from the range 30000–32767, and forwards that traffic to the Service, which then forwards to Pods.

    So traffic flow is: EC2-public-IP:30080 ->  -> nginx Pods

When to Use?
Use NodePort when:
-> you want simple external access in labs
-> you are learning on bare metal / local / EC2
-> no cloud load balancer is being created

Why use it?
Because it is a quick way to access an app from outside the cluster without a separate external load balancer.

### LoadBalancer — step by step

EXAMPLE:

```YAML
apiVersion: v1
kind: Service
metadata:
  name: nginx-lb
  namespace: nginx
spec:
  selector:
    app: nginx
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
```

What it Does:
On a supported cloud platform, Kubernetes asks the cloud provider to provision an external load balancer and attach it to your Service.
creating an external load balancer requires a cloud provider environment.
So the flow: Public Load Balancer IP / DNS -> Service -> Pods

When to Use:
Use Loadbalancer when:
-> running on EKS / AKS / GKE
-> you want public access to your app
-> you want cloud-managed external exposure

Why Use it?
Because it is cleaner and more production-like than manually exposing node ports.

### Headless Service — step by step

EXAMPLE:

```YAML
apiVersion: v1
kind: Service
metadata:
  name: nginx-headless
  namespace: nginx
spec:
  clusterIP: None
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
```

What it Does?
A normal Service gives you one Service IP and load-balances traffic.
A headless Service does not allocate a cluster IP. Instead, DNS resolves the Service name to the set of Pod IPs behind it. Kubernetes DNS docs describe headless Services as resolving to the IPs of all selected Pods rather than to one cluster IP.

It is generally used for SatefulSets..
We have not yet learned it.. we will know more about when studying StatefulSets.

# FULL MENTAL MODEL OF K8S UNTILL NOW:

Now, before proceeding with the next lesson, let us sum up the full scope of our learning and how can we practice this in the real world..

We will see steps for a nodejs backend application (WITHOUT DATABASE 1-tier)

Step 1:

```
  pull a 1 tier application from github
```

Step 2:

```
  cd inside the folder..
```

Step 3: - build the docker image

```
  docker build -t (app-name) .
```

Step 4: Login to dockerhub

```
  docker login
```

Step 5: Tag the image created

```
  docker image tag (image_name):latest (dockerhub_username)/image_name:latest
```

Step 6: Push the same to dockerhub

```
  docker push (dockerhub_username)/image_name:latest
```

Step 7: Create a kind cluster as per initial steps of this chapter and then Create a K8S folder inside the app folder

```
  mkdir -p k8s
  cd k8s
```

Step 8: Create a Namespace
vim namespace.yml
---->>>>

```YAML
kind: Namespace
apiVersion: v1
metadata:
  name: one-tier
```

Step 9: Create a deployment:
vim deployment.yml
----->>>>

```YAML
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nodejs-1-tier-app
  namespace: one-tier
  labels:
    app: nodejs-1-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nodejs-1-app
  template:
    metadata:
      labels:
        app: nodejs-1-app
    spec:
      containers:
      - name: nodejs-1-container
        image: YOUR_DOCKERHUB_USERNAME/nodejs-app:latest
        ports:
        - containerPort: 5000
      volumeMounts:
      - name: app-storage
        mountPath: /usr/src/app/data
    volumes:
    - name: app-storage
      persistentVolumeClaim:
        claimName: nodejs-app-pvc
```

Spet 9: Create PV (Modern way: You only create PVC, K8S automatically creates PV) - REAL WORLD (cloud: You only create: PVC → Kubernetes auto-creates PV)
---->>>>

```YAML
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nodejs-pv
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual-storage
  hostPath:
    path: /mnt/node-data
```

Step 10: Create PVC for app storage (Modern way: You only create PVC, K8S automatically creates PV) - REAL WORLD (cloud: You only create: PVC → Kubernetes auto-creates PV)
---->>>>

```YAML
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nodejs-app-pvc
  namespace: one-tier
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

Step 11: create service.yml
---->>>>

```YAML
kind: Service
apiVersion: v1
metadata:
  name: nodejs-service
  namespace: one-tier
spec:
  selector:
    app: nodejs-1-app
  ports:
    - protocol: TCP
      port: 5000
      targetPort: 5000
  type: ClusterIP
```

Step 12: apply these:

```
kubectl apply -f namespace.yml
kubectl apply -f pv.yml
kubectl apply -f pvc.yml
kubectl apply -f deployment.yml
kubectl apply -f service.yml

```

Step 13: Check resources

```
kubectl get all -n one-tier
kubectl get pvc -n one-tier
```

Step 14: expose the port:

```
sudo -E kubectl port-forward service/nodejs-service -n one-tier 5000:5000 --address=0.0.0.0

```

Step 15: add this port to security group in AWS

# INGRESS:

Ingress is a traffic rule system for incoming HTTP/HTTPS requests into the Kubernetes cluster.

    What if I have many Services?
      if request comes for /, send it to frontend
      if request comes for /api, send it to backend
    That smart entry point is called Ingress.

### HOW DOES IT DIFFER FROM SERVICE?

A Service exposes Pods inside Kubernetes networking or sometimes outside, depending on type.

An Ingress does not directly talk to Pods. It usually works like this:
Internet/User → Ingress → Service → Pod

Ingress is only rules, not the actual traffic engine. An Ingress resource is mainly a YAML object containing routing rules. But rules alone cannot do anything.
So Kubernetes needs something that actually reads those Ingress rules and applies them. That thing is called an: Ingress Controller

To apply Ingress Controller config in kind come outside of the project directory:
kubectl apply -f https://kind.sigs.k8s.io/examples/ingress/usage.yaml

Now check

```
kubectl get ns
```

you will see a ns ingress-nginx

```
kubectl get svc  -n ingress-nginx
```

you will see a svc type loadbalancer, which you will need to expose later..

The YAML object where you write rules:
---->>>>

```YAML
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: frontend-ingress
  namespace: web
spec:
  ingressClassName: nginx
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
```

Suppose you have:
frontend Deployment + Service
backend Deployment + Service

Services:
frontend-service on port 80
backend-service on port 5000

        ------>>>>>>>

```YAML
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
  namespace: web
spec:
  ingressClassName: nginx
  rules:
    - http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: frontend-service
                port:
                  number: 80
          - path: /api
            pathType: Prefix
            backend:
              service:
                name: backend-service
                port:
                  number: 5000
```

# STATEFULSETS:

This is used with databases, as state of the data matters here...
Like: Mysql, MongoDB
this happens so that when a database pod gets deleted, the same state of the new pod is created..

Lets create a new dir mysql..

####EXAMPLE:

We must create service of StatefulSets before the manifest of itself, as we will require the service name in the StatefulSet manifest file..

## Service of statefulsets: it will be headless service: Headless Service (clusterIP: None) is required because StatefulSet needs direct DNS for each Pod.

Example DNS:
mysql-0.mysql-service
mysql-1.mysql-service

Service.yml:

```YAML
kind: Service
apiVersion: v1
metadata:
  name: mysql-service
  namespace: mysql
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
  - name: mysql
    protocol: TCP
    port: 3306
    targetPort: 3306
```

#### Now create the statfulset.yml

```YAML
kind: StatefulSet
apiVersion: apps/v1
metadata:
  name: mysql-statefulset
  namespace: mysql
spec:
  serviceName: mysql-service   ###*****
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
        env:                     ###*****
        - name: MYSQL_ROOT_PASSWORD
          value: root
        - name: MYSQL_DATABASE
          value: devops
        volumeMounts:
        - name: mysql_data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:        ####******
  - metadata:
      name: mysql-data
    spec:
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 1Gi
```

#### WHY VOLUMECLAIMTEMPLATE:

REMEMBER IN DEPLOYMENT WE USED:
volumes:

- name: my-volume
  persistentVolumeClaim:
  claimName: local-pvc

Deployment case: You bring the PVC separately.
Like:
-> first create PVC
-> then Pod/Deployment uses it

So: PVC exists first → Deployment attaches it

#### Why StatefulSet usually uses volumeClaimTemplates?

Because StatefulSet is meant for stateful applications.
StatefulSet creates PVCs → each Pod gets its own storage
If you have 3 MySQL replicas, you usually do not want all of them writing into one same disk carelessly.
You usually want:
mysql-0 gets its own disk
mysql-1 gets its own disk
mysql-2 gets its own disk

That is exactly why volumeClaimTemplates exists.

# CONFIG MAPS:

This is a file that contains all the configurations and env variables of any pod, so that, when any changes are required to be made, it can directly be done from this file...

ConfigMap is used to store non-sensitive configuration data separately from the container image.

### EXAMPLE:

```YAML
kind: ConfigMap
apiVersion: v1
metadata:
  name: mysql-config-map
  namespace: mysql
data:
  MYSQL_DATABASE: devops
```

Note: ConfigMap can also be mounted as a file inside container (like config file).

-> Now in StatefulSets.yml - the value in the env is not direct... change it to valueFrom:

```YAML
env:
- name: MYSQL_ROOT_PASSWORD
  value: root
- name: MYSQL_DATABASE
  valueFrom:
    configMapKeyRef:
      name: mysql-config-map
      key: MYSQL_DATABASE
```

# SECRETS:

Secret stores sensitive data (passwords, tokens), but by default it is only base64 encoded, not encrypted.

Now this is related to the value of the keys in the configmap. This will encode the value and then when used in deployment or statefulsets, it will give extra secutity.. this secret file is converted to binary, which is not decodable easily..

Secrets can be:
✅ env variables
✅ mounted as files

### EXAMPLE:

```YAML
kind: Secret
apiVersion: v1
metadata:
  name: mysql-secret
  namespace: mysql
data:
  MYSQL_ROOT_PASSWORD: paste the value here ##(get this value from bash.. echo (value) | base64)
```

NOW AGAIN EDIT IN THE STATEFULSET in the env:

```YAML
env:
- name: MYSQL_ROOT_PASSWORD
  valueFrom:
    secretKeyRef:
      name: mysql-secret
      key: MYSQL_ROOT_PASSWORD
- name: MYSQL_DATABASE
  valueFrom:
    configMapKeyRef:
      name: mysql-config-map
      key: MYSQL_DATABASE
```

# RESOURCES AND LIMITS

when creating a pod, we should instruct the minimum and maximum limit for scaling it.

### EXAMPLE:

lets take the updated deployment.yml:
here inside the ports, we need to specify the resources... just like in creating pv... but here in the resoucres ->
in requests -> we say our min requiments of cpu and memory
in limits -> we specify our maximum limit of cpu and memory

```YAML
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nginx-deployment
  namespace: nginx
  labels:
    app: nginx
spec:
  replicas: 4
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
        resources:                         ####****
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 200m
            memory: 256Mi
        volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: my-volume
      volumes:
        - name: my-volume
          persistentVolumeClaim:
            claimName: local-pvc
```

# PROBES:

This is used to make sure that a pod is running correctly or not in a port.. This is done by making sure that the pod requests internally on the port no.. Depending on the request it makes:
there are 3 types of probes:
.) Liveness Probe : Checks if the probe is live.. restart container if unhealthy
.) Readiness Probe : Checks if the pod is ready. control traffic (Service will not send traffic if not ready)
.) Startup Probe: Checks if the pod creation has started. delay liveness check until app fully starts

### EXAMPLE:

lets go back to 1 tier app that we did for node js:
here below the container port we test for the readiness/liveness

vim deployment.yml
----->>>>

```YAML
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nodejs-1-tier-app
  namespace: one-tier
  labels:
    app: nodejs-1-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nodejs-1-app
  template:
    metadata:
      labels:
        app: nodejs-1-app
    spec:
      containers:
      - name: nodejs-1-container
        image: YOUR_DOCKERHUB_USERNAME/nodejs-app:latest
        ports:
        - containerPort: 5000
        livenessProbe:                     #####*****
          httpGet:
            path: /
            port: 5000
      volumeMounts:
      - name: app-storage
        mountPath: /usr/src/app/data
    volumes:
    - name: app-storage
      persistentVolumeClaim:
        claimName: nodejs-app-pvc
```

similarly we can create a readiness probe as well:

```YAML
readinessProbe:
      httpGet:
        path: /
        port: 5000
```

now to test if this probe was triggered or not:

```
kubectl get pods -n one-tier
```

copy the pod name and then:

```
kubectl describe pods/(pod_name) -n one-tier
```

# TAINTS/TOLERATIONS:

Taints: this tells the scheduler not to use a tained node..

Toleration: we can add toleration to a tainted node, so that scheduler can schedule pods there..

To Taint:

```
kubectl taint node (node_name) prod=true:NoSchedule
```

To Remove Taint:

```
kubectl taint node (node_name) prod=true:NoSchedule- ##(just add a minus)
```

To Add Toleration:
lets go back to deployment.yml where we were creating a pod (assume node is tained)

vim deployment.yml
----->>>>

```YAML
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nodejs-1-tier-app
  namespace: one-tier
  labels:
    app: nodejs-1-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nodejs-1-app
  template:
    metadata:
      labels:
        app: nodejs-1-app
    spec:
      containers:
      - name: nodejs-1-container
        image: YOUR_DOCKERHUB_USERNAME/nodejs-app:latest
        ports:
        - containerPort: 5000
        livenessProbe:
          httpGet:
            path: /
            port: 5000
      volumeMounts:
      - name: app-storage
        mountPath: /usr/src/app/data
      tolerations:         ####******
        - key: "prod"
          operator: "Equal"
          value: "true"
          effect: "NoSchedule"
    volumes:
    - name: app-storage
      persistentVolumeClaim:
        claimName: nodejs-app-pvc
```

# AUTOSCALING:

The main purpose of k8s was to have pods wich can autoheal and autoscale. When traffic rises, this is what becomes usefull.
there are 3 main types of autoscaling:
.) HPA (horizontal pod autoscaling) .) VPA (vertical pod autoscaling) .) KEDA (Kubernetes event driven autoscaling)

Here you will need to understand the term METRICS: It means number of quantifiable resources (like CPU, RAM usage etc.)
For this you must have a METRICS server installed in the KUBE-SYSTEM ns when kind cluster or any other cluster..

Step1: use the following link to download and install metrics server in kind cluster:

```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Step2: Edit the Metrics server deployment:

```
kubectl -n kube-system edit deployment metrics-server
```

Step3: Add security bypass to deployment under container.args:

```
- --kubelet-insecure-tls
- --kubelet-preferred-address-types=InternalIP,Hostname,ExternalIP
```

Step4: Restart the deployment

```
kubectl -n kube-system rollout restart deployment metrics-server
```

Step5: Verify if Metrics Sever is running:

```
kubectl get pods -n kube-system
```

Now you can check which node is taking most cpu and memory

```
kubectl top nodes
```

you can also check which pod is taking most cpu and memory

```
kubectl top pods -n (name)
```

## HPA:

### HORIZONTAL POD AUTOSCALING:

This is focused on autosacling by creating more replicas.. It is generally used for stateless apps (frontend - backend)

For this let's create another directory like nginx: apache

```
mkdir apache
```

Now create ns:

```YAML
kind: Namespace
apiVersion: v1
metaData:
  name: apache
```

Now create its deployment:

```YAML
kind: Deployment
apiVersion: apps/v1
metadata:
  name: apache-deployment
  namespace: apache
  labels:
    app: apache-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: apache-app
  template:
    metadata:
      labels:
        app: apache-app
    spec:
      containers:
      - name: apache
        image: httpd:latest
        ports:
        - containerPort: 80
        resources:       ####**** (this will be autoscaled)
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 200m
            memory: 256Mi
```

Now create a Service:

```YAML
apiVersion: v1
kind: Service
metadata:
  name: apache-service
  namespace: apache
spec:
  selector:
    app: apache-app
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: ClusterIP
```

##Remember: at any point after creating a service, if you want to access it from outside:

```
curl http://apache-service(svc_name).apache(ns).svc.cluster.local
```

Now if you check with the EC2 public ip on port 80, you should see apache hosted.. it will say "It Works"

NOW CREATE HPA.YML: ### (Note: carefully as this is a bit different than the other manifest files learnt so far....)

```YAML
kind: HorizontalPodAutoscaler
apiVersion: autoscaling/v2
metaData:
  name: apache-hpa
  namespace: apache
spec:
  scaleTargetRef:    ### this is new
    kind: Deployment
    name: apache-deployment
    apiVersion: apps/v1
  minReplicas: 1     ### this is new
  maxReplicas: 5     ### this is new
  metrics:           ### this whole part is new
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 5
```

Now lets forward the port:

```
sudo -E kubectl port-forward service/apache-service -n apache 80:80 --address=0.0.0.0
```

NOW LETS CREATE A LOADGENERATOR TO CREATE LOAD IN THE POD:

We will now create a pod in the name of loadgenerator:

```
kubectl run -i --tty load-generator --image=busybox -n apache /bin/sh
```

Now when the intaractive terminal (it/-i --tty) opens.. run the following command lines:

```
while true; wget -q -O- http://apache-service.apache.svc.cluster.local; done
```

Now it should continiously get the site....

Now open another terminal in the system and do ssh again to to the ec2 instance (as the other one is now busy generating the load and the terminal is unusable..)

Now if you check..

```
kubectl get hpa -n apache
```

you will see that cpu utilization has boomed so the replicas will auto boom up to 5 ..

## VPA:

VPA:
-> does NOT increase replicas
-> increases CPU/memory per Pod

First we will need to download and install the vpa featues from official k8s github page. Follow the instruction given on the page;

https://github.com/kubernetes/autoscaler/blob/master/vertical-pod-autoscaler/docs/installation.md

use this process from the above page:

```
git clone https://github.com/kubernetes/autoscaler.git
cd autoscaler/vertical-pod-autoscaler
git checkout origin/vpa-release-1.0 REGISTRY=registry.k8s.io/autoscaling TAG=1.0.0 ./hack/vpa-process-yamls.sh apply
```

Now lets start from where we ended HPA (assuming all the deployment and service is already created) and delete the hpa and stop the load-generator:

Create vpa.yml

```YAML
kind: VerticalPodAutoscaler
apiVesrsion: autoscaling.k8s.io/v1 ###***
metaData:
  name: apache-vpa
  namespace: apache
spec:
  targetRef:     #### this is not scaleTargetRef
    kind: Deployment
    name: apache-deployment
    apiVersion: apps/v1
  updatePolicy: #### new thing
    updateMode: "Auto" ### other values Initial/Off
```

now:

```
kubectl get vpa -n apache
```

Now forward the port to the public ip:

```
sudo -E kubectl port-forward service/apache-service -n apache 80:80 --address=0.0.0.0
```

now again in one terminal watch this cpu and memory usage

```
watch kubectl get vpa -n apache
```

and in the other terminal run the load-generator...
once the load generator has started, it might take some time, but you will see that the more cpu space will be allocated to the pod as the load increases..

# RBAC: (Role Based Access Control)

In K8S, each cluster can have many users. These users can be bound to a particular role respectively.. this binding between the user and the role is called Role Binding..

When many users or teams are using the same Kubernetes cluster, who is allowed to do what?
That is where RBAC comes in.

RBAC is the permission system of Kubernetes.

It decides:
Who can do something
What they can do
On which resource
In which scope

In Kubernetes, permission is given to a subject. RBAC permissions are usually given to identities OR subjects such as:
-> User
-> Group
-> ServiceAccount

In beginner learning, you will often hear “user”, but inside Kubernetes practice, very commonly permissions are given to ServiceAccounts. Because many applications, Pods, controllers, or tools inside the cluster also need permissions to talk to the Kubernetes API.
-> Human user = you / admin / developer
-> ServiceAccount = an identity used by an app or Pod inside Kubernetes

USERS:
Kubernetes RBAC can work with users, but Kubernetes itself does not fully create and manage normal users the same way it creates ServiceAccounts.
ServiceAccounts are Kubernetes objects. But Users are usually managed by external authentication systems, such as:
.) client certificates
.) identity provider
.) OpenID Connect
.) cloud IAM systems
So Kubernetes can authorize users with RBAC, but often does not itself store them as objects like ServiceAccounts.

GROUPS:
Again, just like User, Group usually comes from an external identity/authentication system.

SERVICE ACCOUNT:
A ServiceAccount is an identity used inside Kubernetes, mostly by:
-> Pods
-> applications running in Pods
-> controllers
-> automation tools inside the cluster
A ServiceAccount is like a Kubernetes identity card for a workload.

REMEMBER: A ServiceAccount is not a Pod. A ServiceAccount is not a Role. A ServiceAccount is not permission itself: It is only an identity.
So remember this carefully:
-> ServiceAccount = who
-> Role = what permission
-> RoleBinding = connection between the two

Where does a ServiceAccount live?
A ServiceAccount is namespace-scoped. That means it belongs to one namespace.

Every namespace automatically gets a ServiceAccount called: default
So if you create a namespace, Kubernetes usually creates: default ServiceAccount in that namespace. And if you create a Pod without specifying any ServiceAccount, the Pod will use that namespace’s default ServiceAccount.

Why not always use the default ServiceAccount?
Because the default ServiceAccount is generic. If many Pods use the same default ServiceAccount, then permissions can become messy and unsafe.

ServiceAccount YAML:

```YAML
kind: ServiceAccount
apiVersion: v1
metaData:
  name: pod-reader-sa
  namespace: dev
```

\*\*\*\*REMEMBER: Kubernetes RBAC = Authorization only, RBAC does not do authetication of any user or group..

#### RESOURCE AND VERBS:

A) Resource
A resource means Kubernetes object.
Examples:
pods
services
deployments
configmaps
secrets
namespaces

B) Verb
A verb means action.
Examples:
get = read one object
list = list many objects
watch = keep watching for changes
create = create object
update = modify existing object
patch = partially modify
delete = remove object

So a permission is basically: verb + resource
Example:
get pods
list services
create deployments
delete pods
This is the heart of RBAC.

### THE 4 MAIN BUILDING BLOCKS OF RBAC:

    -> Role
    -> RoleBinding
    -> ClusterRole
    -> ClusterRoleBinding

1. Role —
   A Role is a set of permissions inside one namespace. A Role is namespace-scoped.
   If you create a Role in namespace dev, it applies only to that namespace, not the whole cluster
   Example:
   A Role may say:
   -> can get pods
   -> can list pods
   -> can watch pods
   But only in namespace dev

2. RoleBinding —
   A Role by itself is just a permission document. It does nothing until attached to someone. That attachment is called: RoleBinding
   And it is also namespace-scoped.
   So RoleBinding usually means: “Give this person (user/group) or service-account this Role in this namespace.”

3. ClusterRole —
   A ClusterRole is similar to a Role, but for the whole cluster. It is not limited to one namespace.
   Examples:
   -> can view nodes
   -> can view namespaces
   -> can manage cluster-wide settings

4. ClusterRoleBinding —
   This binds a ClusterRole to a subject. gives ClusterRole permissions cluster-wide.

**\*** Remember: A RoleBinding can bind:
-> a Role
-> or even a ClusterRole
But the effect of that RoleBinding is still only within that namespace.
So, Binding scope matters a lot.

### MANIFEST FILES:

#### 1) First YAML — Role

Suppose namespace is dev. We want to define a role with permission only to:
-> get pods
-> list pods
-> watch pods

```YAML
kind: Role
apiVersion: rbac.authorization.k8s.io/v1   ##****
metaData:
  name: pod-reader
  namespace: dev
rules:         ####*****
- apiGroups: [""]  ### ** explanation after the yml file..
  verbs: ["get", "list", "watch"]
  resources: ["pods"]
```

Explanation of apiGroups: [""]:
Kubernetes resources belong to API groups.
-> Core resources like: pods, services, configmaps etc.. belong to the core API group, which is written as empty string [""]. This means core resources.

      -> But Deployments belong to apps group, not empty string. ["apps"]

#### 2) Second YAML — RoleBinding

Now let us assign that Role.

Example using a ServiceAccount, so create a SA first:
serviceaccount.yml

```YAML
kind: ServiceAccount
apiVersion: v1
metaData:
  name: dev-user-sa
  namespace: dev
```

rolebinding.yml ---> bind the role to the above created service account....

```YAML
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metaData:
  name: pod-reader-rolebinding
  namespace: dev
subjects:                  ######*****
- kind: ServiceAccount
  name: dev-user-sa
  namespace: dev
roleRefs:                  ######*****
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

explanation of SUBJECTS:
This means: who receives the permission.
explanation of ROLRREFS:
This means: which Role is being assigned.
ALL-IN-ALL MEANING: Give ServiceAccount dev-user the permissions defined in Role pod-reader in namespace dev.

#### 3) ClusterRole -

```YAML
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-reader-cluster
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

#### 4) ClusterRoleBinding -

```YAML
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: pod-reader-cluster-binding
subjects:
- kind: ServiceAccount
  name: dev-user
  namespace: dev
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: pod-reader-cluster
```

#### 5) What about Deployments?

```YAML
rules:
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "watch", "create"]
```

#### 6) Multiple resources in one Role-

```YAML
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-manager
  namespace: dev
rules:
- apiGroups: [""]
  resources: ["pods", "services"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "create", "update"]
```

### How to inspect details :

```
kubectl describe role pod-reader -n dev
kubectl describe rolebinding read-pods-binding -n dev
kubectl describe clusterrole pod-reader-cluster
kubectl describe clusterrolebinding pod-reader-cluster-binding
```

### Very useful command: can-i :

```
kubectl auth can-i get pods -n dev
```

# HELM:

It is a package that helps in accessing all the key values of manifest files in a combined place...
A tool that lets you package, reuse, and manage Kubernetes applications

But we neede to install HELM as its a 3rd party package for K8S..

to Install:
-> Go to this site: https://helm.sh/docs/intro/install/
-> Follow the installation steps...

what is a chart?
A Chart is a package of Kubernetes YAML files.
Think: 👉 Chart = folder containing all your YAML + templates
What is a release ?
A Release is a running instance of a Chart.
Example:
Chart = nginx app template
Release 1 = nginx running in dev
Release 2 = nginx running in prod
What are values?
Values are variables used in templates.
Think of Helm like this:
Chart = blueprint of a house
Values = custom settings (color, size, rooms)
Release = actual built house

## HOW TO USE HELM STEP BY STEP:

### 🚀STEP 1: Create a chart:

```
helm create apache-helm
```

-> You will see a folder (apache-helm) is created inside the helm directory..
-> When you go ionside the folder, you will see
-> charts.yml
-> charts
-> templates
-> values.yml
to check the structure of Helm, you can install another package called "TREE"

```
sudo apt install tree
```

now just write ->

```
tree
```

and you will be able to see the structure of helm..
example:
my-chart/
Chart.yaml
values.yaml
templates/
deployment.yaml
service.yaml
ingress.yaml

So this means that you do not need to write the yml files all by yourself, helm gives you the templates..
for example: when you go inside templates (templating engine) and open deployment.yaml
you can see that who code already written, but with value placeholders..
like:

```YAML
metadata:
  name: {{ include "apache-helm.fullname" . }}
spec:
  replicas: {{ .Values.replicaCount }}
```

You can insert the values in the values.yml file...

NOTE: BUT FOR SAFETY, OPEN ALL THE YAML FILES AND CHECK IF SOMEWHERE THE .VALUE PLACEHOLDER IS MISSING..
for example, sometimes in service.yaml, you might find that targetPort is hardcoded as httpd. In such cases edit it with the .value placeholder.. {{ .Values.service.targetPort }}

Now when inserting the values, open values.yaml ->
edit in the values... that is it...

#### Doubt: I did not see a persistentVolume or pvc manifest file, neither any mention of it in the deployment file... so how can helm help me generate them?

In such a case we need to create a base resuable chart (a Helm chart folder that contains your reusable structure)
for example:
deployment template
service template
optional pvc template
optional ingress template
helper templates

##### Where do I store a reusable Helm chart so another machine can pull it later?

There are three levels of answer, from beginner to real-world.

###### 1) Level 1 — keep the chart in Git

Example mental flow:
git clone <your-repo>
cd base-app-chart
helm install myapp ./ -n dev -f values-dev.yaml
But this method has a limitation: It is source-code reuse, not yet chart distribution. Meaning: you are reusing the folder by cloning code, not by pulling a packaged chart.

###### 2) Level 2 — package the chart and store it in a Helm chart repository

Helm has an official concept called a chart repository.

Step A — create your reusable chart

Step B — package it

```
  helm package base-app-chart
```

This creates something like: base-app-chart-0.1.0.tgz
Helm charts are packaged as versioned tar archives, and chart repositories store these packaged charts plus an index.yaml.

Step C — store that package in a chart repository
The Helm docs say a chart repository can be hosted on things like: Amazon S3, Google Cloud Storage, GitHub Pages, or your own web server

Step D — use it from another machine

```
helm repo add myrepo https://example.com/charts
helm repo update
helm install myapp myrepo/base-app-chart -n dev -f values-dev.yaml
```

Now you are no longer copying folders manually. Now you are pulling a reusable chart from a central place.

###### 3) Level 3 — store charts in an OCI registry

This is the more modern method. The official Helm docs recommend using OCI-based registries to store and share chart packages, and specifically say this is the recommended way if you are considering how to distribute charts. Helm also supports working with container registries that have OCI support.

This is powerful because many teams already use a container registry for Docker images, such as:
Amazon ECR
Azure Container Registry
Google Artifact Registry
Docker Hub
Harbor
Artifactory
So instead of only pushing Docker images there, you can also push Helm charts there.
Step A — package chart:

```
  helm package base-app-chart
```

Step B — log in to registry

```
  helm registry login <your-registry>
```

Step C — push chart

```
  helm push base-app-chart-0.1.0.tgz oci://<your-registry>/helm-charts
```

Step D — install from another machine

```
  helm install myapp oci://<your-registry>/helm-charts/base-app-chart --version 0.1.0 -n dev -f values-dev.yaml
```

### 🚀STEP 2: Now package the helm:

Come out of the helm folder/directory, created by apache-helm and then package the directory..

```
helm package apache-helm
```

### 🚀STEP 3: Now install the chart: (for this lets assume that we need to have 3 different environment of the same app - dev, production, staging)

dev ->

```
helm install dev-apache(env-name) apache-helm(chart-name) -n dev-apache --create-namespace
```

(here i am creating the namespace dev-apache as well, otherwise, this helm would been created in default namespace)

prod->

```
helm install prod-apache apache-helm -n prod-apache --create-namespace
```

stg ->

```
helm install stage-apache apache-helm -n stage-apache --create-namespace
```

UPGRADES TO NEW REVISIONS:
now lets say we need more replicas for production:
-> we can directly change the values.yaml of templates..
-> now open the chart.yaml, upgrade the appVersion to a next figure.. eg.. 1.16.0 -> 1.16.1
-> again package the helm -

```
helm package apache-helm
```

Packaging is needed ONLY when: you are pushing to repo / registry. 👉 NOT required for local upgrade.

-> then:

```
helm upgrade prod-apache ./apache-helm -n prod-apache
```

now if you do:

```
kubectl get pods -n dev-apache
```

-> you will still see old no. of pods
but in prod:

```
kubectl get pods -n prod-apache
```

-> you will see new no. of pods..

DOWNGRADE TO PREVIOUS REVISIONS:
now lets say the prod needs to roll back to old no. of pods:
we can simply rollback the helm to the previous version:

```
helm rollback prod-apache 1(revision no.) -n prod-apache
```

### 🚀STEP 4: check the service name so that you can forward the port in the next step:

```
kubectl get svc -n dev-apache
```

### 🚀STEP 5: forward the port:

```
kubectl port-forward svc/<service-name> -n dev-apache 8000:8000 --address=0.0.0.0 &
```

### 🚀UNINSTALL:

To uninstall deployments:

```
helm uninstall prod-apache -n prod-apache
```

To uninstall helm alltogether at any point:

```
helm uninstall apache-helm
```

# INIT CONTAINERS & SIDECAR CONTAINERS:

Remember: 👉 A Pod can contain multiple containers
👉 WHY multiple containers?
Suppose you have a main app container:
Example:
NodeJS app
Nginx
Backend API
But before starting it, you need:
wait for database
download config file
run migration script
OR alongside it, you need:
logging agent
monitoring agent
proxy
So Kubernetes gives:
👉 Init Containers
👉 Sidecar Containers

## Init Containers — concept:

👉 Init container = runs BEFORE main container
-> runs one by one
-> must complete successfully
-> then main container starts
👉 Init container = setup step
Like:
wait for db
preparing environment

👉 YAML example:

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: init-demo
spec:
  initContainers:       ######*****
  - name: wait-for-db
    image: busybox
    command: ['sh', '-c', 'echo waiting for db; sleep 10']
  containers:
  - name: main-app
    image: nginx
```

## Sidecar Container — concept:

👉 Sidecar = container that runs alongside main container
Example:
Main app:
NodeJS API
Sidecar:
log collector
proxy
metrics exporter

👉 Sidecar = helper assistant

👉 YAML example:

```YAML
apiVersion: v1
kind: Pod
metadata:
  name: sidecar-demo
spec:
  containers:
  - name: main-app
    image: nginx

  - name: log-sidecar
    image: busybox
    command: ['sh', '-c', 'while true; do echo logging; sleep 5; done']
```

👉 Both containers:
share network
share storage (if volumes used)

# Real-world examples:

NOW LET'S BUILD A REALISTIC APP:
👉 NodeJS API + MongoDB + Secret outside Helm + StatefulSet + PVC(statefulsets) + Init Container + Optional Sidecar + HPA + VPA

In real-world Helm:
values.yaml does NOT contain actual passwords.
It only contains:

- secret name
- configuration
  Actual secret values are created outside Helm.
  Deployment reads secrets using:
  valueFrom → secretKeyRef

We are not putting everything into one Pod. We are separating the application into two workloads:
<> MongoDB - It is stateful
-> Secret - The MongoDB credentials will be stored in a Kubernetes Secret created outside Helm
<> NodeJS API
-> Init container
-> Sidecar (optional: Kubernetes documents sidecar logging patterns, but also notes that many applications should log directly to stdout/stderr, with sidecars used for special cases such as file-based logs or log-agent patterns.)

## Full flow before we touch code:

Create namespace
Create external Secret manually
Install Helm chart
Helm creates:
.) MongoDB Service
.) MongoDB headless Service
.) MongoDB StatefulSet
.) NodeJS Deployment
.) API Service
.) optional HPA
.) optional VPA
MongoDB Pod gets stable identity and persistent volume
NodeJS Pod starts
Init container waits for MongoDB Service
NodeJS starts
Optional sidecar starts with NodeJS
HPA/VPA act later based on policy and metrics

### 🧠 STEP 0 — CREATE NS AND SECRET (OUTSIDE HELM)

```
kubectl create namespace dev
```

```
kubectl create secret generic mongo-secret \
  -n dev \
  --from-literal=MONGO_INITDB_ROOT_USERNAME=myuser \
  --from-literal=MONGO_INITDB_ROOT_PASSWORD='StrongPassword123'
```

Verify::

```
kubectl get secret -n dev
kubectl describe secret mongo-secret -n dev
```

### 🧠 STEP 1 — : Install Helm as per notes above and then Helm create chart

```
helm create my-app -n dev
```

-> edit the tree to add the additional templates and edit the templates as per below steps:

📦 HELM CHART STRUCTURE:

```
nodejs-mongo-prod-chart/
  Chart.yaml
  values.yaml
  templates/
    mongodb-headless-service.yaml
    mongodb-statefulset.yaml
    mongodb-service.yaml
    api-deployment.yaml
    api-service.yaml
    hpa.yaml
    vpa.yaml
```

### ⚙️ STEP 2- Chart.yaml

```YAML
apiVersion: v2
name: nodejs-mongo-prod-chart
description: Production-style Helm chart for NodeJS API with MongoDB
type: application
version: 0.1.0
appVersion: "1.0.0"
```

### ⚙️ STEP 3 — values.yaml (CONTROL CENTER):

```YAML
namespace: dev

mongodb:
  name: mongodb
  image: mongo:7
  port: 27017
  database: mydb
  replicaCount: 1
  auth:
    existingSecret: mongo-secret
    usernameKey: MONGO_INITDB_ROOT_USERNAME
    passwordKey: MONGO_INITDB_ROOT_PASSWORD
  persistence:
    enabled: true
    size: 10Gi
    storageClassName: ""
    accessModes:
      - ReadWriteOnce

api:
  replicaCount: 2
  image:
    repository: yourdockerhub/node-api
    tag: latest
    pullPolicy: IfNotPresent
  containerPort: 5000
  service:
    type: ClusterIP
    port: 80
  resources:
    requests:
      cpu: "200m"
      memory: "256Mi"
    limits:
      cpu: "500m"
      memory: "512Mi"

initContainer:
  enabled: true
  image: busybox:1.36

logging:
  mode: stdout   # stdout or file-sidecar
  sidecar:
    enabled: false
    image: busybox:1.36
  logDir: /var/log/app
  logFile: /var/log/app/app.log

hpa:
  enabled: true
  minReplicas: 2
  maxReplicas: 6
  targetCPUUtilizationPercentage: 70

vpa:
  enabled: false
  updateMode: "Off"   # Off, Initial, Auto
  minAllowed:
    cpu: 100m
    memory: 128Mi
  maxAllowed:
    cpu: 1000m
    memory: 1Gi
```

### 📦 STEP 4 — mongodb-headless-service.yaml:

```YAML
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-mongodb-headless
  namespace: {{ .Values.namespace }}
spec:
  clusterIP: None
  selector:
    app: {{ .Release.Name }}-mongodb
  ports:
    - name: mongodb
      port: {{ .Values.mongodb.port }}
      targetPort: {{ .Values.mongodb.port }}
```

### 📦 STEP 5 — mongodb-statefulset.yaml:

```YAML
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: {{ .Release.Name }}-mongodb
  namespace: {{ .Values.namespace }}
spec:
  serviceName: {{ .Release.Name }}-mongodb-headless
  replicas: {{ .Values.mongodb.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}-mongodb
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}-mongodb
    spec:
      containers:
        - name: mongodb
          image: {{ .Values.mongodb.image }}
          ports:
            - containerPort: {{ .Values.mongodb.port }}
              name: mongodb
          env:
            - name: MONGO_INITDB_ROOT_USERNAME
              valueFrom:
                secretKeyRef:
                  name: {{ .Values.mongodb.auth.existingSecret }}
                  key: {{ .Values.mongodb.auth.usernameKey }}
            - name: MONGO_INITDB_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: {{ .Values.mongodb.auth.existingSecret }}
                  key: {{ .Values.mongodb.auth.passwordKey }}
          volumeMounts:
            - name: mongo-data
              mountPath: /data/db
  volumeClaimTemplates:
    - metadata:
        name: mongo-data
      spec:
        accessModes:
          {{- range .Values.mongodb.persistence.accessModes }}
          - {{ . }}
          {{- end }}
        resources:
          requests:
            storage: {{ .Values.mongodb.persistence.size }}
        {{- if .Values.mongodb.persistence.storageClassName }}
        storageClassName: {{ .Values.mongodb.persistence.storageClassName }}
        {{- end }}
```

### 📦 STEP 5 — NodeJS Deployment:

```YAML
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-api
  namespace: {{ .Values.namespace }}
spec:
  replicas: {{ .Values.api.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}-api
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}-api
    spec:
      {{- if .Values.initContainer.enabled }}
      initContainers:
        - name: wait-for-mongodb
          image: {{ .Values.initContainer.image }}
          command:
            - sh
            - -c
            - |
              until nc -z {{ .Release.Name }}-mongodb {{ .Values.mongodb.port }}; do
                echo "Waiting for MongoDB at {{ .Release.Name }}-mongodb:{{ .Values.mongodb.port }}..."
                sleep 2
              done
              echo "MongoDB is reachable"
      {{- end }}

      containers:
        - name: node-api
          image: "{{ .Values.api.image.repository }}:{{ .Values.api.image.tag }}"
          imagePullPolicy: {{ .Values.api.image.pullPolicy }}
          ports:
            - containerPort: {{ .Values.api.containerPort }}
          env:
            - name: PORT
              value: "{{ .Values.api.containerPort }}"
            - name: MONGO_HOST
              value: "{{ .Release.Name }}-mongodb"
            - name: MONGO_PORT
              value: "{{ .Values.mongodb.port }}"
            - name: MONGO_DB
              value: "{{ .Values.mongodb.database }}"
            - name: MONGO_USER
              valueFrom:
                secretKeyRef:
                  name: {{ .Values.mongodb.auth.existingSecret }}
                  key: {{ .Values.mongodb.auth.usernameKey }}
            - name: MONGO_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: {{ .Values.mongodb.auth.existingSecret }}
                  key: {{ .Values.mongodb.auth.passwordKey }}
          resources:
            requests:
              cpu: {{ .Values.api.resources.requests.cpu | quote }}
              memory: {{ .Values.api.resources.requests.memory | quote }}
            limits:
              cpu: {{ .Values.api.resources.limits.cpu | quote }}
              memory: {{ .Values.api.resources.limits.memory | quote }}
          {{- if eq .Values.logging.mode "file-sidecar" }}
          volumeMounts:
            - name: app-logs
              mountPath: {{ .Values.logging.logDir }}
          {{- end }}

        {{- if and (eq .Values.logging.mode "file-sidecar") .Values.logging.sidecar.enabled }}
        - name: log-sidecar
          image: {{ .Values.logging.sidecar.image }}
          command:
            - sh
            - -c
            - |
              touch {{ .Values.logging.logFile }}
              tail -F {{ .Values.logging.logFile }}
          volumeMounts:
            - name: app-logs
              mountPath: {{ .Values.logging.logDir }}
        {{- end }}

      {{- if eq .Values.logging.mode "file-sidecar" }}
      volumes:
        - name: app-logs
          emptyDir: {}
      {{- end }}
```

### 📦 STEP 6 — api-service.yaml:

```YAML
apiVersion: v1
kind: Service
metadata:
  name: {{ .Release.Name }}-api
  namespace: {{ .Values.namespace }}
spec:
  selector:
    app: {{ .Release.Name }}-api
  ports:
    - name: http
      port: {{ .Values.api.service.port }}
      targetPort: {{ .Values.api.containerPort }}
  type: {{ .Values.api.service.type }}
```

### 📦 STEP 7 — hpa.yaml:

```YAML
{{- if .Values.hpa.enabled }}
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: {{ .Release.Name }}-api-hpa
  namespace: {{ .Values.namespace }}
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: {{ .Release.Name }}-api
  minReplicas: {{ .Values.hpa.minReplicas }}
  maxReplicas: {{ .Values.hpa.maxReplicas }}
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: {{ .Values.hpa.targetCPUUtilizationPercentage }}
{{- end }}
```

### 📦 STEP 8 — check for errors:

```
helm lint .
```

### 📦 STEP 9 — Render (generate) all Kubernetes YAML files from my Helm chart — but do NOT deploy them:

```
helm template myapp . -n dev
```

helm template = preview final YAML. Helm replaces:
{{ .Release.Name }} --->>> myapp

### 📦 STEP 10 — Install the release:

```
helm install myapp . -n dev
```

### 📦 STEP 11 — Install the release:

```
helm list -n dev
kubectl get all -n dev
kubectl get pvc -n dev
kubectl get secret -n dev
kubectl get statefulset -n dev
kubectl get deployment -n dev
kubectl get hpa -n dev
kubectl get vpa -n dev
```

---------DONE----------------------------

## How to observe startup:

Watch Pods:

```
kubectl get pods -n dev -w
```

Check init container logs:

```
kubectl logs <api-pod-name> -n dev -c wait-for-mongodb
```

Check app logs:

```
kubectl logs <api-pod-name> -n dev -c node-api
```

Check sidecar logs, only if enabled:

```
kubectl logs <api-pod-name> -n dev -c log-sidecar
```

## How to test the API:

Check the service of the node-js app (not the mongodb):

```
kubectl get svc/api-service -n dev
```

Port forward the svc:

```
kubectl port-forward svc/(svc_name) -n dev 8080:80 --address=0.0.0.0
```

# MONITORING K8S

Observality of K8S can be classified into:
=>Metrics:
Exporter → Prometheus → Grafana

=>Logs:
Promtail → Loki → Grafana

=>Traces:
OpenTelemetry → Jaeger/Grafana Tempo

### Goal:

👉 Detect problems
👉 Debug problems
👉 Predict problems

Node / Pod / K8S Object
->
Exporter / Metrics Endpoint
->
Prometheus Scrapes Metrics
->
Prometheus stores metrics in TSDB
->
Grafana queries Prometheus
->
Dashboards & Graphs

🚀 VERY IMPORTANT WORD:
SCRAPING: Prometheus mainly works using - PULL MODEL. Meaning: 👉 Prometheus goes and FETCHES metrics.

## 🚀 NOW THE NEXT BIG REALIZATION KUBERNETES MONITORING HAS 3 LEVELS

### 1️⃣ Infrastructure Monitoring:

-> Observe:
.) nodes
.) CPU
.) RAM
.) disk

-> Tools:
.) Node Exporter

### 2️⃣ Kubernetes Monitoring

-> Observe:
.) pods
.) deployments
.) restarts
.) HPA

-> Tools:
.) kube-state-metrics

### 3️⃣ Application Monitoring

-> Observe:
.) requests/sec
.) login failures
.) API latency

-> Tools:
.) app metrics endpoint
.) Prometheus client libraries

## Node exporter:

This is a service that takes the info from the nodes and exports it on a particular port

Exports:
OS-level metrics
Node-level metrics

## Kube state Metrics:

Node exporter justs helps in exporing node information... but for components, the information is taken by the defualt namespace (Kube-system) through the kube state metrics..
Exports:
Kubernetes object state

## Goal:

👉 Detect problems
👉 Debug problems
👉 Predict problems

Node / Pod / K8S Object
↓
Exporter / Metrics Endpoint
↓
Prometheus Scrapes Metrics
↓
Prometheus stores metrics in TSDB
↓
Grafana queries Prometheus
↓
Dashboards & Graphs

### 🚀 VERY IMPORTANT WORD: SCRAPING:

SCRAPING: Prometheus mainly works using - PULL MODEL. Meaning: 👉 Prometheus goes and FETCHES metrics.

EXAMPLE:

Suppose Node Exporter exposes metrics on: http://node-exporter:9100/metrics
-> Prometheus periodically visits: /metrics and collects data. This process is called: SCRAPING

Prometheus mainly has 4 jobs:

| Component                | Responsibility         |
| ------------------------ | ---------------------- |
| Scraper                  | Collect metrics        |
| TSDB                     | Store time-series data |
| PromQL                   | Query engine           |
| Alertmanager integration | Trigger alerts         |

Prometheus cannot directly understand everything. So many systems expose metrics using: EXPORTERS. Exporters convert system/application information into Prometheus-readable metrics.

COMMON EXPORTERS

| Exporter           | Purpose                 |
| ------------------ | ----------------------- |
| Node Exporter      | Linux node metrics      |
| kube-state-metrics | Kubernetes object state |
| cAdvisor           | container metrics       |
| MongoDB exporter   | MongoDB metrics         |
| Nginx exporter     | Nginx metrics           |
| MySQL exporter     | MySQL metrics           |

🚨 VERY IMPORTANT FOR YOUR EMS APP :

For your EMS SaaS later, you will eventually monitor:

| Metric                          | Why Important |
| ------------------------------- | ------------- |
| Login requests                  | auth health   |
| API latency                     | performance   |
| DB connections                  | Mongo health  |
| Pod restarts                    | instability   |
| Employee attendance API traffic | load pattern  |

## Prometheus:

This is a data scraper, that takes data from different componets of K8S..
It is a TimeSeries Database that has the following abilities;
-> Scrape Data
-> Query Server that uses PROMQL to query the data
-> Make Graphs
-> Data store

🧠 VERY IMPORTANT REALIZATION: Prometheus does NOT magically understand MongoDB. MongoDB Exporter exposes metrics. Then Prometheus scrapes them.

-> Prometheus stores metrics -> Grafana queries Prometheus -> Grafana visualizes the returned data

### PROMETHEUS OPERATOR:

Managing raw Prometheus configs manually becomes difficult in Kubernetes. So Kubernetes commonly uses: Prometheus Operator
It automates:
-> Prometheus deployment
-> scraping configuration
-> monitoring CRDs

### 🚀 NOW THE NEXT BIG CONCEPT: SERVICE MONITOR

A ServiceMonitor tells Prometheus: 👉 WHICH Services should be scraped.

Application Pod
↓
Service exposes metrics endpoint
↓
ServiceMonitor selects Service
↓
Prometheus Operator updates Prometheus config
↓
Prometheus scrapes metrics

📘 EXAMPLE SERVICE MONITOR:

```YAML
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: nodejs-monitor
  namespace: monitoring
spec:
  selector:
    matchLabels:
      app: nodejs-api
   
```

🧠 WHAT THIS MEANS:

Prometheus should:
-> find Services with label: app=nodejs-api
-> scrape: /metrics
-> every: 15 seconds

🚨 VERY IMPORTANT REAL-WORLD REQUIREMENT: YOUR APP MUST EXPOSE /metrics

REAL NODEJS EXAMPLE TO EXPOSE METRICS:

For NodeJS: Usually library: prom-client

```JavaScript

import client from "prom-client";
import express from "express";

const app = express();

const collectDefaultMetrics = client.collectDefaultMetrics;
collectDefaultMetrics();

app.get("/metrics", async (req, res) => {
  res.set("Content-Type", client.register.contentType);
  res.end(await client.register.metrics());
});

app.listen(5000);
```

### METRICS: WHAT ACTUALLY EXISTS INSIDE /metrics:

📘 EXAMPLE /metrics OUTPUT:

When Prometheus visits: /metrics, it receives plain text like:
http_requests_total 1520
process_cpu_seconds_total 21.5
nodejs_heap_size_total_bytes 68288512

Prometheus parses this text into time-series data.

#### TYPES OF METRICS:

Prometheus mainly uses::

| Metric Type | Purpose                       |
| ----------- | ----------------------------- |
| Counter     | only increases                |
| Gauge       | increases/decreases           |
| Histogram   | request duration distribution |
| Summary     | latency statistics            |

EXAMPLES:

🧠 COUNTER: http_requests_total
Only increases.
Good for: login count, API calls, errors

🧠 GAUGE: memory_usage_bytes
Can: increase/decrease

🧠 HISTOGRAM: request_duration_seconds
Used for: LATENCY
Helps answer: 👉 “How fast is my API?”

#### CUSTOM APPLICATION METRICS:

🚀 NOW THE NEXT BIG REALIZATION: YOUR EMS APP SHOULD EXPOSE BUSINESS METRICS
INFRASTRUCTURE METRICS ≠ BUSINESS METRICS
Infrastructure metric: CPU = 70%, But business metrics are even more important.

EMS BUSINESS METRICS Examples::
employees_logged_in_total
attendance_marked_total
payroll_generation_failures
leave_requests_pending

These are: CUSTOM APPLICATION METRICS

🚨 THIS IS HUGE FOR SAAS PRODUCTS: Because eventually your dashboards should answer: 👉 “Is the business healthy?” NOT just: 👉 “Is CPU healthy?”

📘 NODEJS CUSTOM METRIC EXAMPLE:

```JavaScript
const loginCounter = new client.Counter({
  name: "employees_logged_in_total",
  help: "Total employee logins"
});

app.post("/login", (req, res) => {
  loginCounter.inc();
});
```

Now Prometheus can track: total employee logins

### 🚨 ENDPOINTS:

A Kubernetes Service points to Pods. The actual Pod IPs behind the Service are stored in: Endpoints

EXAMPLE:

Suppose:

```
Service:
  name: ems-api-service
```

selects:

```
app: ems-api
```

Kubernetes automatically creates:

```
Endpoints:
  10.244.1.5:5000
  10.244.1.9:5000
  10.244.2.3:5000
```

These are the actual Pod IPs.

🧠 VERY IMPORTANT REALIZATION: Prometheus ultimately scrapes: ENDPOINTS

📘 CHECK ENDPOINTS:

```
kubectl get endpoints
```

### FULL CHAIN (VERY IMPORTANT)

Pod Labels
↓
Service selects Pods
↓
Endpoints created automatically
↓
ServiceMonitor selects Service
↓
Prometheus discovers Endpoints
↓
Prometheus scrapes metrics

## 🚀 NEXT BIG TOPIC: PROMQL

PromQL = Prometheus Query Language

Used to: query metrics, aggregate metrics, calculate rates, create alerts

### EXAMPLES:

#### 1. CPU usage query:

```
rate(container_cpu_usage_seconds_total[5m])
```

Meaning: 👉 CPU usage rate over last 5 minutes

#### 2. TOTAL REQUESTS

```
http_requests_total
```

#### 3. REQUEST RATE

```
rate(http_requests_total[1m])
```

Meaning: 👉 requests per second

Prometheus stores RAW metrics. PromQL transforms them into:
-> graphs
-> alerts
-> dashboard panels

### 📘 ALERTING:

Prometheus can trigger alerts using: Alertmanager
Examples: CPU > 90%, pod crash looping, MongoDB down

Alertmanager can send: Slack, Email, Teams, PagerDuty

EXAMPLE ALERT FLOW:

Prometheus detects threshold
↓
Alert fires
↓
Alertmanager receives it
↓
Notification sent

ALERTS ARE JUST PROMQL CONDITIONS

Example: CPU > 90%, But internally:

```
expr: rate(container_cpu_usage_seconds_total[5m]) > 0.9
```

#### 📘 SIMPLE ALERT RULE EXAMPLE:

```YAML
groups:
  - name: cpu-alerts
    rules:
      - alert: HighCPUUsage
        expr: rate(container_cpu_usage_seconds_total[5m]) > 0.9
        for: 2m
        labels:
          severity: warning
```

🧠 WHAT for: 2m MEANS:

Condition must remain true for: 2 continuous minutes, before firing alert. This avoids false alarms.

#### 🧠 SO THERE ARE 2 TYPES OF ALERTS:

1️⃣ INFRASTRUCTURE ALERTS: (prebuilt by Helm stack)
2️⃣ APPLICATION / BUSINESS ALERTS: These YOU must create manually.

## Grafana:

This is maily a visulation tool and helps to visualise the data taken from Prometheus..

## STEPS:

-> Make sure helm is installed..

-> Create a new ns monitoring

-> Now add the helm chart of prometheus community..

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
```

-> Update all the charts

```
helm repo update
```

-> Install the Prometheus-Stack

```
helm intsall prometheus-stack prometheus-community/kube-prometheus-stack --namespace monitoring --set prometheus.service.nodePort=30000 --set grafana.service.nodePort=31000 --set grafana.type=NodePort --set prometheus.service.type=NodePort
```

-> to check and get details of ports of prometheus and grafana on nodeport, so that we can use this to forward the port in the next step

```
kubectl get pods -n monitoring
kubectl get svc -n monitoring
```

-> port-forward

for prometheus

```
kubectl port-forward svc/(service_name_of_prometheus) -n monitoring (port_no.):(port_no.) --address=0.0.0.0 &
```

similarly for grafana

```
kubectl port-forward svc/(service_name_of_grafana) -n monitoring (port_no.):(port_no.) --address=0.0.0.0 &
```

-> expose the ports in aws...

-> paste the public ip of ec2 instance on address bar and then open port of port no..

-> when prometheus opens, there is no need to enter the scraping configurations...as we have used helm, the configurations are already there..
Go to status -> Targets..

-> when grafana opens, it will ask for usernae and password...
username is admin
to get the password:

```
kebectl get secret prometheus-stack-grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 decode
```

-> now here no need to connect prometheus as a data source ---> it is already added as we used helm --->
just go to -> connections -> datasource -> you will see prometheus is already added -> just click on "build a dashboard" -> "add visualisation" -> select the data source of prometheus ->
now in the metrics you can select which ever metrics you need..
in the label filters -> you can select the namespace

## POINTS TO NOTE FOR REAL WORLD DEPLOYMENT MONITORING: CUSTOM EMS APPLICATION MONITORING

👉 If you install: kube-prometheus-stack, from the Prometheus Community Helm chart, then:
MANY ALERTS ARE ALREADY PRE-BUILT, You do NOT have to write everything manually from zero.

🧠 WHAT THE HELM CHART ALREADY GIVES YOU:

| Alert                 | Example               |
| --------------------- | --------------------- |
| Node down             | server unreachable    |
| Pod crash looping     | frequent restarts     |
| High CPU              | CPU threshold crossed |
| Memory pressure       | low available memory  |
| Disk pressure         | disk almost full      |
| Kubernetes API issues | API server unhealthy  |
| HPA issues            | autoscaling problems  |

These are called: DEFAULT ALERT RULES

### 🚨 BUT… YOUR BUSINESS ALERTS ARE NOT INCLUDED: (as stated earlier in notes)

For your EMS app later:
The Helm chart will NOT know:
-> employee login failures
-> payroll API issues
-> attendance system failures
-> invoice processing problems

#### 🚀 VERY IMPORTANT COMPONENT: PrometheusRule

Custom alerts are usually created using: PrometheusRule CRD

EXAMPLE:

```YAML
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: ems-alerts
  namespace: monitoring
spec:
  groups:
    - name: ems.rules
      rules:
        - alert: HighLoginFailures
          expr: rate(login_failures_total[5m]) > 10
          for: 2m
          labels:
            severity: critical
```

🧠 WHAT HAPPENS HERE:
Prometheus continuously checks: rate(login_failures_total[5m]) > 10 ===> If true for: 2 minutes => then:👉 alert fires.

🚨 VERY IMPORTANT REALIZATION: Installing monitoring stack ≠ fully monitored application.

The Helm chart gives: FOUNDATION
You later add: BUSINESS INTELLIGENCE

#### FOR YOUR EMS APP:

| Alert                       | Why                       |
| --------------------------- | ------------------------- |
| Mongo connections high      | DB overload               |
| Attendance API latency high | slow response             |
| Payroll job failures        | critical business failure |
| Login failure spike         | possible auth issue       |
| Pod restart increase        | instability               |

But first remember this:
-> ServiceMonitor only collects metrics.
-> PrometheusRule creates alerts from metrics.
-> Grafana dashboard visualizes metrics.
And your EMS app must expose metrics like:
/metrics

Example metrics your NodeJS app/exporters should expose:
-> ems_attendance_request_duration_seconds
-> ems_payroll_job_failures_total
-> ems_login_failures_total
-> mongodb_connections
-> kube_pod_container_status_restarts_total

##### 1. ServiceMonitor for EMS API:

```YAML
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: ems-api-servicemonitor
  namespace: monitoring
  labels:
    release: prometheus-stack
spec:
  namespaceSelector:
    matchNames:
      - dev
  selector:
    matchLabels:
      app: ems-api
  endpoints:
    - port: http
      path: /metrics
      interval: 15s
```

Important: Your EMS API Service must have:

```YAML
metadata:
  labels:
    app: ems-api
spec:
  ports:
    - name: http
      port: 80
      targetPort: 5000
```

##### 2. PrometheusRule for EMS Alerts

```YAML
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: ems-custom-alerts
  namespace: monitoring
  labels:
    release: prometheus-stack
spec:
  groups:
    - name: ems.rules
      rules:

        - alert: MongoConnectionsHigh
          expr: mongodb_connections > 400
          for: 5m
          labels:
            severity: warning
            app: ems
          annotations:
            summary: "MongoDB connections are high"
            description: "MongoDB connections have been above 400 for 5 minutes."

        - alert: AttendanceApiLatencyHigh
          expr: histogram_quantile(0.95, rate(ems_attendance_request_duration_seconds_bucket[5m])) > 1.5
          for: 3m
          labels:
            severity: critical
            app: ems
          annotations:
            summary: "Attendance API latency is high"
            description: "95% of attendance API requests are taking more than 1.5 seconds."

        - alert: PayrollJobFailures
          expr: increase(ems_payroll_job_failures_total[10m]) > 0
          for: 1m
          labels:
            severity: critical
            app: ems
          annotations:
            summary: "Payroll job failure detected"
            description: "At least one payroll job failed in the last 10 minutes."

        - alert: LoginFailureSpike
          expr: rate(ems_login_failures_total[5m]) > 5
          for: 2m
          labels:
            severity: warning
            app: ems
          annotations:
            summary: "Login failure spike"
            description: "Login failures are increasing rapidly. Possible auth issue or brute-force attempt."

        - alert: EMSPodRestartIncrease
          expr: increase(kube_pod_container_status_restarts_total{namespace="dev", pod=~"ems-api.*"}[10m]) > 2
          for: 2m
          labels:
            severity: warning
            app: ems
          annotations:
            summary: "EMS API pod restart increase"
            description: "EMS API pods restarted more than 2 times in the last 10 minutes."
```

EVEN IF YOU MUST CREATE BUSINESS ALERTS AND SERVICE MONITORS MANUALLY, BUT:
✅ You do NOT have to create a Grafana dashboard ConfigMap:
You can do this manually in Grafana UI:
Grafana
→ Connections / Data sources
→ Prometheus already added
→ Build dashboard
→ Add visualization
→ Write/select PromQL query
→ Save dashboard

But later on you should use ConfigMap for Grafana, So later with ArgoCD:
Git dashboard YAML
↓
ArgoCD applies it
↓
Grafana gets dashboard automatically

##### 3. Grafana Dashboard ConfigMap: This is a simple starter dashboard.

```YAML
apiVersion: v1
kind: ConfigMap
metadata:
  name: ems-grafana-dashboard
  namespace: monitoring
  labels:
    grafana_dashboard: "1"
data:
  ems-dashboard.json: |
    {
      "title": "EMS Application Dashboard",
      "panels": [
        {
          "title": "Login Failures Rate",
          "type": "timeseries",
          "targets": [
            {
              "expr": "rate(ems_login_failures_total[5m])"
            }
          ]
        },
        {
          "title": "Attendance API P95 Latency",
          "type": "timeseries",
          "targets": [
            {
              "expr": "histogram_quantile(0.95, rate(ems_attendance_request_duration_seconds_bucket[5m]))"
            }
          ]
        },
        {
          "title": "Payroll Job Failures",
          "type": "timeseries",
          "targets": [
            {
              "expr": "increase(ems_payroll_job_failures_total[1h])"
            }
          ]
        },
        {
          "title": "MongoDB Connections",
          "type": "timeseries",
          "targets": [
            {
              "expr": "mongodb_connections"
            }
          ]
        },
        {
          "title": "EMS Pod Restarts",
          "type": "timeseries",
          "targets": [
            {
              "expr": "increase(kube_pod_container_status_restarts_total{namespace=\"dev\", pod=~\"ems-api.*\"}[10m])"
            }
          ]
        }
      ],
      "schemaVersion": 39,
      "version": 1
    }
```

Very Important Note: These YAML files only work if the metric names actually exist.

So the chain is:

EMS NodeJS app exposes /metrics
↓
Service exposes EMS app
↓
ServiceMonitor tells Prometheus to scrape it
↓
Prometheus stores metrics
↓
PrometheusRule creates alerts
↓
Grafana dashboard shows graphs
