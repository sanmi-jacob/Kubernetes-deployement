# Kubernetes-deployement
Install minikube and Hyperkit
Before start building our containerized application, we need to create a local dev environment. minikube is a great tool that allows us to implement multi-node Kubernetes clusters locally on our machines and comes with kubectl already installed.
Install minikube on your system however you see fit, but for me (macOS), the Homebrew package manager was the most straightforward method and also allows us to install the virtual machine environment we’ll need.
# brew install minikube

# Cluster setup
If you remember from Docker Swarm, our container workloads run on ‘nodes,’ which can be physical or virtual machines/servers that provide high availability and fault tolerance. In Swarm, nodes are assigned roles of manager or worker. The principle is the same in Kubernetes, but nodes are split into roles of the control plane — which contain all the master nodes — and workers. Let’s set up our cluster in minikube:
One (1) master/control plane node.
Three (3) worker nodes.
By default, minikube will assign our local machine as a master/control plane node and all others as workers.
# minikube start --driver hyperkit --nodes 4
# Now, we can use kubectl commands to get our nodes
# kubectl get nodes

We only want pods to be scheduled on worker nodes, so we need to add special instructions to the control plane in the form of a taint. A Taint allows a node to repel a set of pods by telling the scheduler not to schedule any pods unless they have a matching toleration.

# kubectl taint nodes node_name key1=value1:NoSchedule 

# Deploy the frontend web tier
Similar to Docker Compose, an application’s entire infrastructure can be deployed using a single YAML file; however, Kubernetes YAML files can be a little more involved and detailed. Although a single YAML file can be used, the deployment configurations might be easier to manage if we break them out into a few organized files.
We’ll start with the frontend web tier and deploy our pods that contain our Apache web servers.

# Frontend web Service
We’ve been working with Docker Swarm so, you might be familiar with the term, ‘services.’ In Docker, a service is the image or image type run on a container, such as Apache, NGINX, Node, Postgres, etc… It is essentially a description and role of a container/task that will be deployed in a cluster.
A Kubernetes service refers to a network and stable endpoint/address for pods to either communicate internally or with outside services. There are three main types of services: ClusterIP, NodePort, and LoadBalancer.
A ClusterIP creates virtual IPs to allow pods to communicate with each other. A default ClusterIP is given to our cluster, so the control plane and worker nodes can communicate with each other.

# Frontend web deployment
Now we’re ready to create a Deployment! Deployments are more akin to Docker services. It’s the set of instructions and specifications for the pods including the image, number of replicas, ports, restart policies, etc…
# As we can see, pod deployments are under the kind: Deployment, with its own API Group. We should also notice the selector section where we've given it a matchlabel of the same name as the web-httpd-service selector, so they know to connect to each other.
Under template, we can provide the container information, such as the name, image, and port.
# Deploy the pods and service
We’re ready to deploy our first pods! This is the easy part. All we have to do is apply the configuration and point to our YAML file.
# As we can see, pod deployments are under the kind: Deployment, with its own API Group. We should also notice the selector section where we've given it a matchlabel of the same name as the web-httpd-service selector, so they know to connect to each other.
Under template, we can provide the container information, such as the name, image, and port.
Deploy the pods and service
We’re ready to deploy our first pods! This is the easy part. All we have to do is apply the configuration and point to our YAML file.

# kubectl apply -f ./frontend/frontend-kube-app.yml
We can get a node's IP by running kubectl get nodes -o wide.

# Deploy the backend application tier
Now that we have our first tier, out of the way, it’s pretty much rinse-and-repeat — but with varying specs to fit your needs.
Since we don’t have any application source code or need to connect anything at this point, we’ll just create a simple Deployment for the app backend. In the backend directory, create a new YAML file called backend-kube-app.yml.
# kubectl apply -f ./backend/backend-kube-app.yml

# Deploy the backend database tier
We’re ready for the final phase, and deploy the database (Postgres) for our app. The database can be a little more involved so let’s break down what we’ll need:
A ConfigMap to store secrets such as usernames and passwords.
A PersistantVolume and PersistantVolumeClaim to define and allocate storage for the A database.
A Service to connect the database
A Deployment to define and deploy the database pod to our cluster.
# Create a ConfigMap
It’s always best practice to separate data from code. A good way to do this in development is through a ConfigMap. Kubernetes has its own API group for this purpose.
In the database directory, create a new YAML file called postgres-config.yml

# kubectl apply -f ./database/postgres-config.yml

# Create a PersistantVolume & PersistantVolumeClaim
Since pods are ephemeral — meaning once they’re gone, so is all the data they carried — we’ll need persistent storage in place for our database. Here, we’ll define the capacity, access, and the hostPath of the volume.
Once we create the volume, we need a PersistantVolumeClaim, which defines how the users request and consume PV resources.
Create a new YAML file named postgres-pvc-pv.yml.
# Apply the PV and PVC to the cluster.
# kubectl apply -f ./database/postgres-pvc-pv.yml

# Create the Service
For now, we’ll just create a simple NodePort service and expose port 5432.
Create a new YAML file named database-kube-app.yml.
# Create the Deployment
Finally, we’ll create the deployment options for our Postgres database (in the same file).
# kubectl apply -f ./database/postgres-pvc-pv.yml
# Test database connection
For good measure, let’s test and see if we can connect to the database.
# kubectl exec -it [pod-name] --  psql -h localhost -U admin --password -p 5432 postgresdb
# Enter the DB password and type \l to list all the databases.