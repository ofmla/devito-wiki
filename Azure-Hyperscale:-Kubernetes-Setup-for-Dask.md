by Qie Zhang, Microsoft Azure Global (collaboration with the Devito team)

This wiki shows step by step how to set up and deploy an Azure [Kubernetes](https://kubernetes.io/) cluser for running a seismic imaging job in parallel through [Azure Kubernetes Service (AKS)](https://azure.microsoft.com/en-us/services/kubernetes-service/), where AKS automates application deployment, scaling and management. This document is inherited and expanded from [Kubernetes setup](https://github.com/devitocodes/daks/wiki/Kubernetes-setup) on Devito github.

The deployment of Kubernetes cluster returns an IP address that is taken by [Dask](https://dask.org/) which distributes workloads (such as calculating the FWI gradient for each shot) to parallel workers in the Kubernetes cluster. The Kubernetes setup can be used for 2D/3D seismic RTM/FWI jobs - here is a [2D FWI example](https://github.com/devitocodes/devito/blob/master/examples/seismic/tutorials/04_dask.ipynb) related to this document.

1. Install Azure command-line interface (Azure CLI) that is a set of commands used to create and manage Azure resources. ([more info](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-apt?view=azure-cli-latest))
```
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

2. Sign in with Azure CLI. If the CLI can open your default browser, it will do so and load an Azure sign-in page. ([more info](https://docs.microsoft.com/en-us/cli/azure/authenticate-azure-cli?view=azure-cli-latest))
```
az login
```
After sign-in, you may want to list all subscriptions and set a subscription to be the current active subscription - this is optional. ([more info](https://docs.microsoft.com/en-us/cli/azure/account?view=azure-cli-latest))
```
az account list --output table
az account set --subscription "YOUR-SUBSCRIPTION-TO-BE-CURRENT"
```

3. Create a new resource group. ([more info](https://docs.microsoft.com/en-us/cli/azure/group?view=azure-cli-latest))
```
az group create --name fwirg --location southcentralus
```
To confirm the creation, you can list all resource groups.
```
az group list --output table
```

4. Create an Azure Container Registry (ACR). ([more info](https://docs.microsoft.com/en-us/cli/azure/acr?view=azure-cli-latest#az-acr-create)) 
```
az acr create --resource-group fwirg --name fwiacr --sku Basic
```

5. You can install [Docker](https://www.docker.com/) as below, but suggest instead to follow the Docker [official instruction](https://docs.docker.com/engine/install/ubuntu/) to install Docker.
```
sudo apt-get update
sudo apt-get remove docker docker_engine docker.io
sudo apt install docker.io
```

6. Log in to the Azure Container Registry through the Docker CLI. ([more info](https://docs.microsoft.com/en-us/cli/azure/acr?view=azure-cli-latest#az-acr-login))
```
sudo az acr login --name fwiacr
```
After the successful login, you should expect the message `#Login Succeeded`.

7. List container registries in your resource group and show the results in a table. ([more info](https://docs.microsoft.com/en-us/cli/azure/acr?view=azure-cli-latest#az-acr-list))
```
az acr list --resource-group fwirg --query "[].{acrLoginServer:loginServer}" --output table
```
The output should look like below which will be used in step 10.
```
#AcrLoginServer
#-----------------
#fwiacr.azurecr.io
```

8. Download the Devito repo.
```
git clone https://github.com/devitocodes/devito.git
```

9. In the Devito home directory, create the Devito docker image. Note this is only for CPU (GPU capability will be added later). ([more info](https://docs.docker.com/engine/reference/commandline/build/))
```
sudo docker build -t devito_base . -f docker/Dockerfile
```

10. Locally, tag the Devito docker image. Note `fwiacr.azurecr.io` is from step 7. ([more info](https://docs.docker.com/engine/reference/commandline/tag/))
```
sudo docker tag devito_base fwiacr.azurecr.io/devito_base:v1
```

11. Upload the Devito docker image to the Azure Container Registry. ([more info](https://docs.docker.com/engine/reference/commandline/push/))
```
sudo docker push fwiacr.azurecr.io/devito_base
```
List repositories in the  Azure Container Registry to confirm the success of the command above. ([more info](https://docs.microsoft.com/en-us/cli/azure/acr/repository?view=azure-cli-latest#az-acr-repository-list))
```
az acr repository list --name fwiacr --output table
```
The output should look like below.
```
#Result
#-----------
#devito_base
```

12. Create a new Kubernetes cluster. You can replace the VM [Standard_HB120rs_v2](https://docs.microsoft.com/en-us/azure/virtual-machines/hbv2-series) with the one you prefer. Standard_HB120rs_v2 has 480G memory which is a good choice for 3D FWI. `--node-count 2` denotes 2 VMs to be used for the cluster. ([more info](https://docs.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest#az-aks-create))
```
az aks create \
     --resource-group fwirg \
     --name fwicluster1 \
     --node-vm-size=Standard_HB120rs_v2 \
     --node-count 2 \
     --generate-ssh-keys \
     --attach-acr fwiacr
```
List Kubernetes clusters to confirm the success of the command above. ([more info](https://docs.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest#az-aks-list))
```
az aks list --output table
```

13. Download and install kubectl, the Kubernetes command-line interface (CLI). ([more info](https://docs.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest#az-aks-install-cli))
```
sudo az aks install-cli
```

14. Get access credentials for the Kubernetes cluster. ([more info](https://docs.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest#az-aks-get-credentials))
```
az aks get-credentials --resource-group fwirg --name fwicluster1
```

15. Create the file `dask-cluster.yaml`. The content of the yaml file is listed at the end of this document.

16. Apply the Kubernetes configuration. ([more info](https://kubectl.docs.kubernetes.io/pages/app_management/apply.html))
```
kubectl apply -f dask-cluster.yaml
```
This will setup a Kubernetes cluster with 1 Dask scheduler with an open port 8786 to the world, and 16 workers that will connect to this scheduler automatically. Run the command below to confirm all pods are created. ([more info](https://kubernetes.io/docs/tasks/access-application-cluster/list-all-running-container-images/))
```
kubectl get pods
```
 
17. Find the IP address of the scheduler service. ([more info](https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/))
```
kubectl get services
``` 
The output looks like below. Keep down the EXTERNAL-IP of the LoadBalancer, which will be used in the Dask configuration in step 19.
```
NAME            TYPE           CLUSTER-IP     EXTERNAL-IP     PORT(S)          AGE
devito-server   LoadBalancer   10.0.169.103   40.80.209.180   8786:31150/TCP   85m
kubernetes      ClusterIP      10.0.0.1       <none>          443/TCP          101m
```

18. Open up the Devito [Dask tutorial](https://github.com/devitocodes/devito/blob/master/examples/seismic/tutorials/04_dask.ipynb) in a Jupyter notebook (you can choose to use docker or not). Note this Dask tutorial notebook is a 2D FWI example that does not require much memory, so you do not have to go to Standard_HB120rs_v2 (which is a good choice for 3D FWI).

19. In cell 5 of the notebook, replace the two lines 
```
cluster = LocalCluster(n_workers=nsources, death_timeout=600)
client = Client(cluster)
```
with 
```
client = Client('40.80.209.180:8786')
```
where `40.80.209.180` is the EXTERNAL-IP of the scheduler/LoadBalancer we found in step 17. 


20. Run all cells in the notebook. It will distribute jobs to 16 workers on the Kubernetes cluster we created.

21. When the job is finished, run the command to delete the Kubernetes cluster. ([more info](https://docs.microsoft.com/en-us/cli/azure/aks?view=azure-cli-latest#az-aks-delete))
```
az aks delete --resource-group fwirg --name fwicluster1
```

22. If you would also like to delete the resource group, run this command. ([more info](https://docs.microsoft.com/en-us/cli/azure/group?view=azure-cli-latest#az-group-delete))
```
az group delete --name fwirg
```

&nbsp;

File `dask-cluster.yaml` content is listed below, as used in step 15.
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: devito-server
spec:
  replicas: 1
  selector:
    matchLabels:
      app: devito-server
  template:
    metadata:
      labels:
        app: devito-server
    spec:
      nodeSelector:
        "beta.kubernetes.io/os": linux
      containers:
      - name: devito-server
        image: fwiacr.azurecr.io/devito_base:v1
        command: ['/venv/bin/dask-scheduler']
        ports:
        - containerPort: 8786
          name: devito-server
---
apiVersion: v1
kind: Service
metadata:
  name: devito-server
spec:
  type: LoadBalancer
  ports:
  - port: 8786
  selector:
    app: devito-server
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: devito-worker
spec:
  replicas: 16
  selector:
    matchLabels:
      app: devito-worker
  strategy:
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
  minReadySeconds: 5 
  template:
    metadata:
      labels:
        app: devito-worker
    spec:
      nodeSelector:
        "beta.kubernetes.io/os": linux
      containers:
      - name: devito-worker
        env:
        - name: PYTHONPATH
          value: /app
        - name: DEVITO_LANGUAGE
          value: "openmp"
        - name: OMP_PROC_BIND
          value: "TRUE"
        image: fwiacr.azurecr.io/devito_base:v1
        command: ['/venv/bin/dask-worker', 'tcp://devito-server:8786']
        ports:
        - containerPort: 80
```