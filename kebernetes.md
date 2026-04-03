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

            Esc :wq enter

      #Step 2:
        inside the folder where you would like to create this cluster:
            kind create cluster --name (name the cluster) --config=config.yml

        to check:
          kubectl cluster-info --context kind-(cluster name)

        to ckeck the nodes in the cluster:
          kubectl get nodes --context kind-(cluster name)

        to set deafult a particular cluster:
          kubectl config use-context kind-(cluster name)

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
      kubectl get namespace
            or
      kubectl get ns

    the namespace is very important, as there may be pods running in certain name sapces and may be no pods running at all... so to check:
      kubectl get pods -n (name of namespace...)

    # create a ns:
      kubectl create ns (name)
        eg.. kubectl create ns nginx

    Default namespaces exist:
        default
        kube-system
        kube-public
        kube-node-lease

# PODS:

    to create a pod:
      kubectl run (pod_name) --image=(DOCKER_image_name) -n (ns_name)
        eg: kubectl run nginx --image=ngnix -n nginx

    to check a pod:
      kubectl get pods -n (ns_name)
        eg.. kubectl get pods -n nginx

# Create everything through a MANIFEST file:

    till now we were using command lines... now we will learn about all these recources through a .yml (Manifest) file...

    # creating ns through manifest file
        -> vim namespace.yml
        ->
            kind: Namespace
            apiVersion: v1
            metadata:
              name: nginx
        ->
          esc :wq enter

        -> kubectl apply -f namespace.yml

    # creating pods through manifest file:
        -> vim pod.yml
        ->
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

        -> esc :wq enter
        -> kubectl apply -f pod.yml

        -> to get inside any pod with interactive terminal:
              kubectl exec -it nginx-pod -n nginx -- bash

        -> to get detailed reports of pods:
              kubectl describe pod/nginx-pod -n nginx

# DEPLOYMENT:

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
            eg...  taking a back up, updating, patching etc..

      -> CRON Jobs: these are tasks that run on a schedule created by us or the system..
          for this we should know about CRON (schedule)
          *(min 0-59) *(hour 0-23) *(day of month 1-31) *(month 1-12) *(day of week 0-6 or names)
          A CronJob object uses one cron-format schedule string in .spec.schedule. Kubernetes describes a CronJob as one line of a Unix crontab and uses cron format in the schedule field.

          Explanation:
            Think of cron as a calendar rule.
                eg..
                1. 0 9 * * * (Meaning of *: every possible value for that
                              field)
                  means: you are saying:
                  minute = 0
                  hour = 9
                  day of month = every day
                  month = every month
                  day of week = every day of week
                  in other words: Run every day at 9:00 AM

                2. * * * * *     means: every minute
                3. 0 0 * * *     means: Every day at midnight
                4. 30 18 * * *   means: Every day at 6:30 PM
                5. 0 2 * * 0     means: Every Sunday at 2:00 AM
                6. 15 1 1 * *    means: On the 1st day of every month at
                                        1:15 AM
                Read left to right:


    EXPLANATION OF EACH OF THE ABOVE BY EXAPMLES:

      For the REPLICATION CONTROLLER to identify the pod that will be created, we must give the following in the manifest file when creating deployemnt:
      -> LABEL : we must provide a label to the pod
      -> SELECTOR: we must provide a selector to the deployment to select the labeled pod

          the logic is, when the label name and selector name match, the pod is replicated..



      # creating deployment of the pods through manifest file:

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

      1. DEPLOYMENT:
                -> vim deploment.yml
                ->
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

                -> esc :wq enter

                -> check: kubectl get pods -n nginx
                      and you will see 4 pods ready....

                  ## HOW DOES IT HELP TO SCALE UP?
                      in times of high traffic.. you can simply do:
                          kubectl scale deployemnt/nginx-deployment -n ngnix --replicas=10
                              you will see 10 pods ready straight away...

                  ###NOTE: at any point of time if you want a descriptive pod details:
                              kubectl get pods -n nginx -o wide
                            this will show you which worker node is scheduled for which pod etc..

                  ## HOW DOES DEPLYMENT HELP WITH ROLL OUT UPDATES:

                        Lets say we were working with nginx version 1.26.2 and now we want to update it to nginx version 1.27.3... in such a case we will run:

                          kubectl set image deployment/nginx-deployment -n nginx ngnix=nginx:1.27.3

                            as soon as we do this, we will see that few pods are rolling updates, wheras others are still on the previous vesrions...
                              only when the previous updating pods are fully updated, they will be up, and then the rest will get updated...

                            this way, the service will never be down...

      2. REPLICASET:
                  -> vim replicaset.yml
                  ->
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

                  -> esc :wq enter

                  -> check: kubectl get pods -n nginx
                      and you will see 4 pods ready....

      3.DAEMONSET:
                  -> vim daemonset.yml
                  ->
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

                  -> esc :wq enter

                  -> check: kubectl get pods -n nginx
                      and you will see 3 pods ready... as there are 3 worker nodes...

      4. JOBS:

                  This is different from the other sets, as it is a job that we want to run with our instructions...

                  here in SPEC, we need parameters like:
                    COMPLETIONS: how many successful runs you want.
                    PARALLELISM: how many Pods may run at the same time.
                    TEMPLATE:
                      METADATA: this follows same structure as other sets above.. change the names accordingly..
                      SPEC:
                      -name: batch-container
                       image: busybox:latest  ****************
                            (this is a container whose function is just to run command.... just like nginx function is to serve the web pages)
                       command: ["sh", "-c", "echo hello world && sleep 10"]
                            (this means run a command in shell and then sleep after 10 seconds)
                      RESTART POLICY: Never
                            (if the job fails or the work is done, would you like to restart this?)


                   Example:
                    -> vim jobs.yml
                    ->
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
                                command: ["sh", "-c", "echo hello world &&
                                        sleep 10]
                            restartPolicy: Never

                      -> esc :wq enter

                      -> kubectl apply -f job.yml

                      -> check: kubectl get jobs -n nginx
                          you will see the pod state as running for first 10s and then change the state to complete after 10s
                             - from here we also learn that pods have differnet states eg. container creating, running, terminating, completed etc..

                      -> check logs: kubectl logs pod/demo-job-qqrh(pod_name) -n nginx
                          you will see: hello world..

        5.  CRONJOBS:

              A CronJob is not the Pod itself.
                The actual chain is: CronJob → Job → Pod

                  CronJob decides when
                  Job decides run this task now
                  Pod actually runs the container

              very important partS of CRONJOB is the SCHEDULE (when to run the job).. and the JOB-TEMPLATE that defines the job.

              Basic CronJob YAML structure:

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

# STORAGE:

      Persistent Volumes (PV): A volume is storage that Kubernetes makes available to the Pod.
      Volume Mount: A volume mount is the place where that storage box appears inside the container.
      Persistent Volume Claim (PVC): A Pod does not directly ask for a disk in most beginner setups. Instead it says: I need storage of this size. That request is the PVC. Then Kubernetes connects that PVC to actual storage.

      PERSISTENT VOLUME (PV):
            the whole idea is to attach (persist) the data inside the pod with the host machine  (PV = storage resource in Kubernetes cluster (it may be local disk, EBS, NFS, cloud storage etc.))  so that when a pod is lost, then the new pod created should get this data from the host machine/EBS/CLOUD. this storage inside the host machine/others.. is PV.
              We gebrally allocate such volumes manually. lets say if our host machine is 30gb, so we say to kubernetes, pesist a volume of 1Gi for my pod data.

          WHAT IS THE NEED OF VOLUMES:
            Inside Kubernetes, a Pod is temporary. So, when the Pod dies, the files inside that container also disappear.
              Don't Confuse:  Kubernates deployment will definetely create another pod as per replicas set earler, as soon as a pod die, but the data inside that pod is not bound to any volume yet. So the data inside that pod gets lost.

          Example yml of PV:

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
                persistentVolumeReclaimPolicy: Retain # This will help in rtaining this
                                                                      volume
                storageClassName: local-storage #this clould be cloud-storage,
                                                              network-storage
                hostPath:
                  path: /mnt/data

            now after applying.. when you do
              kubectl get pv ...
                you will see, 1Gi is available

      PERSISTENT VOLUME CLAIM (PVC):

            Now after creating the PV, we need to claim it for our use. So we need PVC.

          Example PVC yml:

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

            now after applying, when you do:
              kubectl get pv
                you will see that there is a 1Gi pv, but the status will say bound...

            VERY IMPORTANT — PV ↔ PVC MATCHING:
              You must understand this clearly: PVC requests → PV provides
              Matching happens based on::
                | Field            | Must match         |
                | ---------------- | ------------------ |
                | storageClassName | ✅                 |
                | size             | PVC ≤ PV           |
                | accessModes      | must be compatible |

            Example from previous case:
              PV:
                storage: 1Gi
                storageClassName: local-storage
              PVC:
                storage: 1Gi
                storageClassName: local-storage
              👉 So Kubernetes binds them

              IMPORTANT REALITY YOU MUST KNOW:
                👉 This study notes are using: hostPath      - ONLY for learning ❗
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

                  make sure yoiu dlete the previous deployemnt... then edit the deployment.yml ->

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
                          type: ClusterIP #(other values: NodePort, Loadbalancer etc.,
                                              Headless) *** FOR DWESCRIPTION SEE BELOW

        Now after apply, if you want to cheack ALL (pods, deploment, service):
              kubectl get all -n ngnix

        Problem: this cluster is in a docker container..so we need to forward the port to access pods from outside:
            sudo -E kubectl port-forward service/ngnix-service -n nginx 80:80 --address=0.0.0.0
            now make sure that you expose the port 80 on ec2 instance

              But remember:
                    kubectl port-forward is mainly for testing/debugging
                    it is not the normal production exposure method


        ## SERVICE TYPES:
                ClusterIP, NodePort, LoadBalancer, and ExternalName
                  (A headless Service is created by setting clusterIP: None; it is a Service pattern rather than a separate type value.)

          ###ClusterIP — step by step:
            EXAMPLE:
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

            What it Does?
              A normal Service gives you one Service IP and load-balances traffic.
                A headless Service does not allocate a cluster IP. Instead, DNS resolves the Service name to the set of Pod IPs behind it. Kubernetes DNS docs describe headless Services as resolving to the IPs of all selected Pods rather than to one cluster IP.

              It is generally used for SatefulSets..
                We have not yet learned it.. we will know more about when studying StatefulSets.

# FULL MENTAL MODEL OF K8S UNTILL NOW:

          Now, before proceeding with the next lesson, let us sum up the full scope of our learning and how can we practice this in the real world..

            We will see steps for a nodejs backend application (WITHOUT DATABASE 1-tier)

            Step 1:
              pull a 1 tier application from github
            Step 2:
              cd inside the folder..
            Step 3: - build the docker image
              docker build -t (app-name) .
            Step 4: Login to dockerhub
              docker login
            Step 5: Tag the image created
              docker image tag (image_name):latest (dockerhub_username)/image_name:latest
            Step 6: Push the same to dockerhub
              docker push (dockerhub_username)/image_name:latest
            Step 7: Create a K8S folder inside the app folder
              mkdir -p k8s
              cd k8s
            Step 8: Create a Namespace
              vim namespace.yml
                  ---->>>>
                    kind: Namespace
                    apiVersion: v1
                    metadata:
                      name: one-tier

            Step 9: Create a deployment:
              vim deployment.yml
                  ----->>>>
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

              Spet 9: Create PV (Modern way: You only create PVC, K8S automatically creates PV) - REAL WORLD (cloud: You only create: PVC → Kubernetes auto-creates PV)
                      ---->>>>
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

              Step 10: Create PVC for app storage (Modern way: You only create PVC, K8S automatically creates PV) - REAL WORLD (cloud: You only create: PVC → Kubernetes auto-creates PV)
                      ---->>>>
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

              Step 11: create service.yml
                      ---->>>>
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

              Step 12: apply these:
                kubectl apply -f namespace.yml
                kubectl apply -f pv.yml
                kubectl apply -f pvc.yml
                kubectl apply -f deployment.yml
                kubectl apply -f service.yml

              Step 13: Check resources
                kubectl get all -n one-tier
                kubectl get pvc -n one-tier

              Step 14: expose the port:
                sudo -E kubectl port-forward service/nodejs-service -n one-tier 5000:5000 --address=0.0.0.0

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
          kubectl get ns
            you will see a ns ingress-nginx
          kubectl get svc  -n ingress-nginx
            you will see a svc type loadbalancer, which you will need to expose later..

          The YAML object where you write rules:
            ---->>>>
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

            Suppose you have:
              frontend Deployment + Service
              backend Deployment + Service

            Services:
                frontend-service on port 80
                backend-service on port 5000

                    ------>>>>>>>
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

# STATEFULSETS:

      This is used with databases, as state of the data matters here...
           Like: Mysql, MongoDB
           this happens so that when a database pod gets deleted, the same state of the new pod is created..

        Lets create a new dir mysql..

        ####EXAMPLE:

        We must create service of StatefulSets before the manifest of itself, as we will require the service name in the StatefulSet manifest file..

        ## Service of statefulsets: it will be headless service

                    kind: Service
                    apiVersion: v1
                    metadata:
                      name: mysql-service
                      namespace: mysql
                    spec:
                      clusterIP: none
                      selector:
                        matchLabels:
                          app: mysql
                      ports:
                      - name: mysql
                        protocol: TCP
                        port: 3306
                        targetPort: 3306

      #### Now create the statfulset.yml

                    kind: StatefulSets
                    apiVersion: apps/v1
                    metadata:
                      name: mysql-statefulset
                      namespace: mysql
                    spec:
                      serviceName: mysql-service   *****
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
                          accessModes: "ReadWriteOnce"
                          resources:
                            requests:
                              storage: 1Gi

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

        ### EXAMPLE:
            kind: ConfigMap
            apiVersion: v1
            metadata:
              name: mysql-config-map
              namespace: mysql
            data:
              MYSQL_DATABASE: devops

          -> Now in StatefulSets.yml

              the value in the env is not direct... change it to valueFrom:

                env:
                - name: MYSQL_ROOT_PASSWORD
                  value: root
                - name: MYSQL_DATABASE
                  valueFrom:
                    configMapKeyRef:
                      name: mysql-config-map
                      key: MYSQL_DATABASE

# SECRETS:

     Now this is related to the value of the keys in the configmap. This will encode the value and then when used in deployment or statefulsets, it will give extra secutity.. this secret file is converted to binary, which is not decodable easily..

      ###EXAMPLE:

          kind: Secret
          apiVersion: v1
          metadata:
            name: mysql-secret
            namespace: mysql
          data:
            MYSQL_ROOT_PASSWORD: paste the value here ##(get this value from bash.. echo
                                                           (value) | base64)

          NOW AGAIN EDIT IN THE STATEFULSET in the env:

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

# RESOURCES AND LIMITS

        when creating a pod, we should instruct the minimum and maximum limit for scaling it.
        ###EXAMPLE:
            lets take the updated deployment.yml:
              here inside the ports, we need to specify the resources... just like in creating pv... but here in the resoucres ->
                          in requests -> we say our min requiments of cpu and memory
                          in limits -> we specify our maximum limit of cpu and memory

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

# PROBES:

    This is used to make sure that a pod is running correctly or not in a port.. This is done by making sure that the pod requests internally on the port no.. Depending on the request it makes:
        there are 3 types of probes:
          .) Liveness Probe : Checks if the probe is live
          .) Readiness Probe : Checks if the pod is ready
          .) Startup Probe: Checks if the pod creation has started

      ###EXAMPLE:
          lets go back to 1 tier app that we did for node js:
            here below the container port we test for the readiness/liveness

               vim deployment.yml
                  ----->>>>
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

        similarly we can create a readiness probe as well:
                        readinessProbe:
                              httpGet:
                                path: /
                                port: 5000

        now to test if this probe was triggered or not:
          kubectl get pods -n one-tier
        copy the pod name and then:
          kubectl describe pods/(pod_name) -n one-tier

      ***********BUT NOTE: It will fail in this case, as this is running on ClusterIP

# TAINTS/TOLERATIONS:

      ###Taints: this tells the scheduler not to use a tained node..
      ###Toleration: we can add toleration to a tainted node, so that scheduler can      schedule pods there..
      To Taint:
        kubectl taint node (node_name) prod=true:NoSchedule
      To Remove Taint:
        kubectl taint node (node_name) prod=true:NoSchedule- ##(just add a minus)

      To Add Toleration:
          lets go back to deployment.yml where we were creating a pod (assume node is tained)

                  vim deployment.yml
                  ----->>>>
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
                          tolerations:
                            - key: "prod"
                              operator: "Equal"
                              value: "true"
                              effect: "NoSchedule"
                        volumes:
                        - name: app-storage
                          persistentVolumeClaim:
                            claimName: nodejs-app-pvc

# AUTOSCALING:

    The main purpose of k8s was to have pods wich can autoheal and autoscale. When traffic rises, this is what becomes usefull.
    there are 3 main types of autoscaling:
      .) HPA (horizontal pod autoscaling)  .) VPA (vertical pod autoscaling) .) KEDA (Kubernetes event driven autoscaling)

      Here you will need to understand the term Metrics: It means number of quantifiable resources (like CPU, RAM usage etc.)
        For this you must have a METRICS node installed in the KUBE-SYSTEM ns when kind cluster or any other cluster..

    Step1: use the following link to download and isntall metrics server in kind cluster:

      kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

    Step2: Edit the Metrics server deployment:

      kubectl -n kube-system edit deployment metrics-server

    Step3: Add security bypass to deployment under container.args:

      - --kubelet-insecure-tls
      - --kubelet-preferred-address-types=InternalIP,Hostname,ExternalIP

    Step4: Restart the deployment

      kubectl -n kube-system rollout restart deployment metrics-server

    Step5: Verify if Metrics Sever is running:

      kubectl get pods -n kube-system

    Now you can check which node is taking most cpu and memory
      kubectl top nodes
    you can also check which pod is taking most cpu and memory
      kubectl top pods -n (name)

# HPA:

    ### HORIZONTAL POD AUTOSCALING:

          This is focused on autosacling by creating more replicas.. It is generally used for stateless apps (frontend - backend)

        For this lets create another directory like nginx : apache
            mkdir apache

        Now create ns:

                        kind: Namespace
                        apiVersion: v1
                        metadata:
                          name: apache
