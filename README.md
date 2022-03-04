# .NET Core worker processing Azure Service Bus Queue scaled by KEDA
A simple Docker container written in .NET that will receive messages from a Service Bus queue and scale via KEDA.

The message processor will receive a single message at a time (per instance), and sleep for 2 second to simulate performing work. When adding a massive amount of queue messages, KEDA will drive the container to scale out according to the event source (Service Bus Queue).

We provide samples for the following scenarios:

- [**Process Azure Service Bus Queue by using Azure AD Pod Identity**](pod-identity.md)
- [**Process Azure Service Bus Queue by using connection string authentication**](connection-string-scenario.md)

![Scenario](images/scenario.png)

> 💡 *If you want to learn how to scale this sample with KEDA 1.0, feel free to read about it [here](https://github.com/kedacore/sample-dotnet-worker-servicebus-queue/tree/keda-v1.0).*

## Kubernetes setup

```sh
REGION_NAME=australiaeast
RESOURCE_GROUP=aks-demos
SUBNET_NAME=aks-subnet
AKS_CLUSTER_NAME=aks-demo-$RANDOM
VERSION=1.22.6
NODE_COUNT=3
ADD_ONS="monitoring"

# Create a basic AKS cluster
az aks create \
--resource-group $RESOURCE_GROUP \
--name $AKS_CLUSTER_NAME \
--node-count $NODE_COUNT \
--location $REGION_NAME \
--kubernetes-version $VERSION \
--enable-addons $ADD_ONS \
--enable-managed-identity \
--generate-ssh-keys

# Install the Kubernetes CLI tools and get an authenticated configuration for the cluster
az aks install-cli
az aks get-credentials --name $AKS_CLUSTER_NAME --resource-group $RESOURCE_GROUP

# List Kubernetes nodes to verify connection to the cluster
kubectl get nodes

# Install Helm 3 CLI (see: https://helm.sh/docs/intro/install/)

# Mac/Linux
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh

# Windows (via chocolatey)
choco install kubernetes-helm

# Install KEDA for pod autoscaling based on events
helm repo add kedacore https://kedacore.github.io/charts
helm repo update

kubectl create namespace keda
helm install keda kedacore/keda --version 2.0.0 --namespace keda
```
