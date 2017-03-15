# Creating a Kubernetes Cluster 

This scenario provides instructions for the following tasks:
- Create a Kubernetes cluster with one worker node
- Install the CLIs for using the Kubernetes API and managing Docker images
- Create a private registry to store your container images

## Lession 1. Setting up the CLI

Install the IBM Bluemix Container Service CLI, the image registry CLI, and their prerequisites. These CLIs are used in later lessons and are required to manage your Kubernetes cluster from your local machine, create images to deploy as containers, and in a later tutorial, deploy apps into the cluster. 

1. If you do not have one yet, create a [Bluemix account](https://console.ng.bluemix.net/registration/). Make note of your user name and password as this information is required later.


2. As a prerequisite for the Bluemix CLI, install the [Cloud Foundry CLI](https://github.com/cloudfoundry/cli/releases). You must install the Cloud Foundry CLI in the default location for your operating system, otherwise the PATH environment variable does not match your installation directory. The prefix for running the Cloud Foundry CLI commands is `cf`.


3. As a prerequisite for the Bluemix Kubernetes plug-in, install the [Bluemix CLI](http://clis.ng.bluemix.net/ui/home.html). The prefix for running commands by using the Bluemix CLI is `bx`.


4. Log into the Bluemix CLI. 
    ```bash
    $ bx login
    ```

5. Follow the prompts to select an account and space to log in to.


6. To create Kubernetes clusters, and manage worker nodes, install the Bluemix Kubernetes plug-in. The prefix for running commands by using the Bluemix Kubernetes plug-in is `bx cs`. 
    ```bash
    $ bx plugin repo-add stage https://plugins.stage1.ng.bluemix.net
    $ bx plugin install cs -r stage
    ```

7. Initialize the Bluemix Kubernetes plug-in. 
	```bash
	$ bx cs init --host https://us-south.containers.bluemix.net
	```

8. To view a local version of the Kubernetes dashboard and to deploy apps into your clusters, install the [Kubernetes CLI](https://kubernetes.io/docs/user-guide/prereqs/). The prefix for running commands by using the Kubernetes CLI is kubectl.

	**a. Download the Kubernetes CLI.**
    
	OS X: http://storage.googleapis.com/kubernetes-release/release/v1.5.1/bin/darwin/amd64/kubectl
    
    Linux: http://storage.googleapis.com/kubernetes-release/release/v1.5.1/bin/linux/amd64/kubectl

	Windows: http://storage.googleapis.com/kubernetes-release/release/v1.5.1/bin/windows/amd64/kubectl.exe
    
    **b. For OSX and Linux users, convert the binary file to an executable.**
    
    ```bash
    $ chmod +x kubectl
	```
    Make sure that /usr/local/bin is listed in your PATH system variable. The PATH variable contains all directories where your operating system can find executable files. The directories that are listed in the PATH variable serve different purposes. /usr/local/bin is used to store executable files for software that is not part of the operating system and that was manually installed by the system administrator. 

	```bash
    $ echo $PATH
	/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin
	```
	Move the executable file to the /usr/local/bin directory
    ```bash
    $ mv kubectl /usr/local/bin/kubectl
	```
    
9. To manage a private image registry with IBM Bluemix Container Registry Service, install the Bluemix Container Registry plug-in. The prefix for running Bluemix Containers Registry commands is `bx cr`. 
	```bash
    $ bx plugin install container-registry -r Bluemix
	```
    To verify that the plug-ins are installed properly, run the following command:
	```bash
    $ bx plugin list
	```

10.	To build images locally and push them to your private image registry, install the [Docker CE CLI](https://www.docker.com/community-edition#/download). If you are using Windows 8 or earlier, you can install the [Docker Toolbox](https://www.docker.com/products/docker-toolbox) instead.

Congratulations! You successfully created your Bluemix account and installed the CLIs for the following lessons and tutorials. Next, access your cluster by using the CLI.

## Lesson 2: Setting up your cluster environment

1. Create your free Kubernetes cluster
    ```bash
    $ bx cs cluster-create --name wordpress
	```
    A free cluster comes with one worker node to deploy container pods upon. A worker node is the compute host, typically a virtual machine, that your pods run on. An app in production runs replicas of a pod across multiple worker nodes to provide higher availability for your app.
    
> **Note:** It can take up to 15 minutes for the worker node machine to be ordered, and for the cluster to be set up and provisioned. 


2. In the meantime, log in to the IBM Bluemix Container Registry CLI. 
	```bash
    $ bx cr login
	```
3. Set up your own private image registry in Bluemix to securely store and share Docker images with all cluster users. A private registry in Bluemix is identified by a namespace that you set in this step. The namespace is used to create a unique URL to your private registry that developers can use to access private Docker images. You can create multiple namespaces in your organization to group and organize your images. For example, you can create a namespace for every department, environment, or app.

    In this example, the wordpress wants to create only one private registry in Bluemix, so they choose wordpress as their namespace to group all images in their organization. Replace <your_namespace> with a namespace of your choice. For now, choose a namespace that means something to you, rather than something that is related to the tutorial.
	```bash
    $ bx cr namespace-add <your_namespace>
	```
    
    Consider the following rules when you choose a namespace for your organization.
    - Your namespace must be unique in Bluemix. 
    - Your namespace can be 4 - 30 characters long.
    - Your namespace must start with at least one letter or number.
    - Your namespace can contain lowercase letters, numbers, or underscores (_) only. 

4. Before you continue to the next step, verify that the deployment of your worker node is complete. 
	```bash
    $ bx cs workers wordpress
    ID                                           Public IP       Private IP    Machine Type  State     Status   
    dal10-pa8dfcc5223804439c87489886dbbc9c07-w1  169.47.223.113  10.171.42.93  free      	deployed  Deploy Automation Successful   
	```
    
5. Set the context for your cluster in your CLI. Every time you log in to the IBM Bluemix Container Service CLI to work with the wordpress, you must run these commands to set the path to the cluster's configuration file as a session variable. The Kubernetes CLI uses this variable to find a local configuration file and certificates that are necessary to connect with the cluster in Bluemix.
    
    a. Download the configuration file and certificates for the pr_firm_cluster cluster. 
    ```bash
    $ bx cs cluster-config wordpress
    export KUBECONFIG=/Users/ibm/.bluemix/plugins/cs-cli/clusters/wordpress/kube-config-dal10-wordpress.yml
	```
	b. Copy and paste the command from the previous step to set the KUBECONFIG environment variable and configure your CLI to run kubectl commands against your cluster. 
    
6. Verify that both Kubernetes secrets were created in your cluster namespace. Every Bluemix service is defined by environment variables that are called VCAP_SERVICES. VCAP_SERVICES include confidential information about the service, such as the user name, password and URL that the container uses to access the service. To securely store this information, Kubernetes secrets are used. In this example, one secret includes the credentials for accessing the instance of the Watson Tone Analyzer that is provisioned in your Bluemix account and one secret includes credentials for the Cloudant service. 
	```bash
    $ kubectl get secrets --namespace=default
    NAME                       TYPE                                  DATA      AGE
    bluemix-default-secret     kubernetes.io/dockercfg               1         1h
    default-token-kf97z        kubernetes.io/service-account-token   3         1h
	```
Great work! The cluster is created, configured, and your local environment is ready for you to start deploying apps into the cluster.