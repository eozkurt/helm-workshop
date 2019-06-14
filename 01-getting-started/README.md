

# Create AKS Cluster

In this lab we will create our Azure Kubernetes Services (AKS) distributed compute cluster.

## Prerequisites

* Azure Account

## Instructions

1. Login to Azure Portal at http://portal.azure.com.

2. Open the Azure Cloud Shell and choose Bash Shell (do not choose Powershell)

3. The first time Cloud Shell is started will require you to create a storage account.


5. Ensure you are using the correct Azure subscription you want to deploy AKS to.
    ```
    # View subscriptions
    az account list
    ```
    ```
    # Verify selected subscription
    az account show
    ```

    ```
    # Set correct subscription (if needed)
    az account set --subscription <subscription_id>

    # Verify correct subscription is now set
    az account show
    ```

6. Create Azure Service Principal to use through the labs

    ```bash
    az ad sp create-for-rbac --skip-assignment
    ```
    This will return the following. !!!IMPORTANT!!! - Please copy this information down as you'll need it for labs going forward.

    ```bash
    "appId": "7248f250-0000-0000-0000-dbdeb8400d85",
    "displayName": "azure-cli-2017-10-15-02-20-15",
    "name": "http://azure-cli-2017-10-15-02-20-15",
    "password": "77851d2c-0000-0000-0000-cb3ebc97975a",
    "tenant": "72f988bf-0000-0000-0000-2d7cd011db47"
    ```

    Set the values from above as variables **(replace <appId> and <password> with your values)**.

    DON'T MESS THIS STEP UP. REPLACE THE VALUES IN BRACKETS!!!

    ```bash
    # Persist for Later Sessions in Case of Timeout
    APPID=<appId>
    echo export APPID=$APPID >> ~/.bashrc
    CLIENTSECRET=<password>
    echo export CLIENTSECRET=$CLIENTSECRET >> ~/.bashrc
    ```

7. Create a  unique identifier suffix for resources to be created in this lab.

    ```bash
    UNIQUE_SUFFIX=$USER$RANDOM
    # Remove Underscores and Dashes (Not Allowed in AKS and ACR Names)
    UNIQUE_SUFFIX="${UNIQUE_SUFFIX//_}"
    UNIQUE_SUFFIX="${UNIQUE_SUFFIX//-}"
    # Check Unique Suffix Value (Should be No Underscores or Dashes)
    echo $UNIQUE_SUFFIX
    # Persist for Later Sessions in Case of Timeout
    echo export UNIQUE_SUFFIX=$UNIQUE_SUFFIX >> ~/.bashrc
    ```

    *** Note this value as it will be used in the next couple labs. ***

8. Create an Azure Resource Group in East US.

    ```bash
    # Set Resource Group Name using the unique suffix
    RGNAME=aks-rg-$UNIQUE_SUFFIX
    # Persist for Later Sessions in Case of Timeout
    echo export RGNAME=$RGNAME >> ~/.bashrc
    # Set Region (Location)
    LOCATION=eastus
    # Persist for Later Sessions in Case of Timeout
    echo export LOCATION=eastus >> ~/.bashrc
    # Create Resource Group
    az group create -n $RGNAME -l $LOCATION
    ```

9. Create your AKS cluster in the resource group created above with 3 nodes. We will check for a recent version of kubnernetes before proceeding. We are also including the monitoring add-on for Azure Container Insights. You will use the Service Principal information from step 5.

    Use Unique CLUSTERNAME

    ```bash
    # Set AKS Cluster Name
    CLUSTERNAME=aks${UNIQUE_SUFFIX}
    # Look at AKS Cluster Name for Future Reference
    echo $CLUSTERNAME
    # Persist for Later Sessions in Case of Timeout
    echo export CLUSTERNAME=aks${UNIQUE_SUFFIX} >> ~/.bashrc
    ```  

    Get available kubernetes versions for the region. You will likely see more recent versions in your lab.

    ```bash
    az aks get-versions -l $LOCATION --output table

    KubernetesVersion    Upgrades
    -------------------  -----------------------
    1.12.5               None available
    1.12.4               1.12.5
    1.11.7               1.12.4, 1.12.5
    1.11.6               1.11.7, 1.12.4, 1.12.5
    1.10.12              1.11.6, 1.11.7
    1.10.9               1.10.12, 1.11.6, 1.11.7
    1.9.11               1.10.9, 1.10.12
    1.9.10               1.9.11, 1.10.9, 1.10.12
    ```

    Set the version to one with available upgrades (in this case v 1.12.4)

    ```bash
    K8SVERSION=1.12.4
    ```

    > The below command can take 10-20 minutes to run as it is creating the AKS cluster. Please be PATIENT and grab a coffee...

    ```bash
    # Create AKS Cluster
    az aks create -n $CLUSTERNAME -g $RGNAME \
    --kubernetes-version $K8SVERSION \
    --service-principal $APPID \
    --client-secret $CLIENTSECRET \
    --generate-ssh-keys -l $LOCATION \
    --node-count 3 \
    --enable-addons monitoring \
    --no-wait
    ```

10. Verify your cluster status. The `ProvisioningState` should be `Succeeded`
    ```bash
    az aks list -o table
    ```

    ```bash
    Name                 Location    ResourceGroup         KubernetesVersion    ProvisioningState    Fqdn
    -------------------  ----------  --------------------  -------------------  -------------------  -------------------------------------------------------------------
    ODLaks-v2-gbb-16502  eastus   ODL_aks-v2-gbb-16502  1.11.4                Succeeded odlaks-v2--odlaks-v2-gbb-16-b23acc-17863579.hcp.centralus.azmk8s.io
    ```

11. Get the Kubernetes config files for your new AKS cluster

    ```bash
    az aks get-credentials -n $CLUSTERNAME -g $RGNAME
    ```

12. Verify you have API access to your new AKS cluster

      > Note: It can take 5 minutes for your nodes to appear and be in READY state. You can run `watch kubectl get nodes` to monitor status.

     ```bash
     kubectl get nodes
     ```
     
     ```bash
     NAME                       STATUS    ROLES     AGE       VERSION
     aks-nodepool1-16622101-0   Ready     agent     24m       v1.11.4
     aks-nodepool1-16622101-1   Ready     agent     24m       v1.11.4
     aks-nodepool1-16622101-2   Ready     agent     24m       v1.11.4
     ```
 
     To see more details about your cluster:

     ```bash
     kubectl cluster-info
     ```

     ```bash
     Kubernetes master is running at https://aksbrian13-aks-rg-brian1327-471d33-09e2e91f.hcp.eastus.azmk8s.io:443
     Heapster is running at https://aksbrian13-aks-rg-brian1327-471d33-09e2e91f.hcp.eastus.azmk8s.io:443/api/v1/namespaces/kube-system/services/heapster/proxy
     KubeDNS is running at https://aksbrian13-aks-rg-brian1327-471d33-09e2e91f.hcp.eastus.azmk8s.io:443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
     kubernetes-dashboard is running at https://aksbrian13-aks-rg-brian1327-471d33-09e2e91f.hcp.eastus.azmk8s.io:443/api/v1/namespaces/kube-system/services/kubernetes-dashboard/proxy
     Metrics-server is running at https://aksbrian13-aks-rg-brian1327-471d33-09e2e91f.hcp.eastus.azmk8s.io:443/api/v1/namespaces/kube-system/services/https:metrics-server:/proxy
     ```

     You should now have a Kubernetes cluster running with 3 nodes. You do not see the master servers for the cluster because these are managed by Microsoft. The Control Plane services which manage the Kubernetes cluster such as scheduling, API access, configuration data store and object controllers are all provided as services to the nodes.


## Installing the Helm Client

1. Initialize Helm

    Helm helps you manage Kubernetes applications — Helm Charts helps you define, install, and upgrade even the most complex Kubernetes application. Helm has a CLI component and a server side component called Tiller. 
    * Initialize Helm and Tiller:

        ```bash
        kubectl apply -f ~/kubernetes-hackfest/labs/helm-setup-deploy/rbac-config.yaml
        helm init --service-account tiller --upgrade
        ```

    * Validate the install (the Helm version may be newer in your lab):
        ```bash
        helm version
        ```

        ```bash
        Client: &version.Version{SemVer:"v2.12.1", GitCommit:"20adb27c7c5868466912eebdf6664e7390ebe710", GitTreeState:"clean"}
        Server: &version.Version{SemVer:"v2.12.2", GitCommit:"20adb27c7c5868466912eebdf6664e7390ebe710", GitTreeState:"clean"}
        ```

        > Note: It can take a minute or so for Tiller to start

```console
$ helm version
```

On Mac, the easiest way to install or upgrade Helm is to use Homebrew.

If you need to install helm:

```console
$ brew install kubernetes-helm
```

If you need to upgrade helm:

```console
$ brew upgrade kubernetes-helm
```

On Ubuntu:

```console
$ sudo snap install helm --classic
```

And on Windows:

```console
$ choco install kubernetes-helm
```

> More installation methods are documented in the [documentation for installing
> Helm](https://docs.helm.sh/using_helm/#installing-helm)

## Initialize Helm and Install Tiller

Once you have Helm ready, you can initialize the local environment and install
Tiller into your Kubernetes cluster in one step.

```console
$ helm init
```

Helm will use whatever Kubernetes cluster your context is pointing to. Use
`kubectl` to learn about your contexts:

```console
$ kubectl config current-context
docker-for-desktop
```

You can verify your setup by running the version command.

```console
$ helm version
Client: &version.Version{SemVer:"v2.14.0", GitCommit:"05811b84a3f93603dd6c2fcfe57944dfa7ab7fd0", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.14.0", GitCommit:"05811b84a3f93603dd6c2fcfe57944dfa7ab7fd0", GitTreeState:"clean"}
```

Inspect the deployment if you get an error.

```console
$ kubectl -n kube-system describe deployment tiller-deploy
```

Once Tiller is up and running, you'll need to grant an admin role to the default
service account.

```console
$ kubectl create clusterrolebinding add-on-cluster-admin \
  --clusterrole=cluster-admin --serviceaccount=kube-system:default
```

IMPORTANT: Please note that this is ok for personal development, but for
production use cases, defining the exact roles and permissions on your cluster
requires a deeper knowledge of Kubernetes' RBAC model


Up next: [Installing Charts](../02-installing-charts/).
