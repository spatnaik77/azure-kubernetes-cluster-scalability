# azure-kubernetes-cluster-scalability
# Introduction
To keep up with application demands in Azure Kubernetes Service (AKS), you may need to adjust the number of nodes that run your workloads. AKS supports scaling at both pod level and cluster level. Using Pod autoscaler, you can scale the pod instances in a cluster. Cluster autoscaler watches the pods and when its not possible to schedule new pods due to lack of computing resources, it adds nodes to the cluster.<br>
![aks-cluster-autoscaler](https://github.com/spatnaik77/azure-kubernetes-cluster-scalability/blob/master/diagrams/aks-cluster-autoscaler.png)
<hr>
In this POC we will do the following:
* We will use this docker - https://cloud.docker.com/u/spatnaik77/repository/docker/spatnaik77/helloindia . It exposes a REST API which returns "hello india"
* Create an AKS cluster with min node as 3 and maximum node as 10. 
* Create a deployment and load balancer to expose the rest APIs
* Create Pod autoscaler
* Create a performance test and generate enough load on the cluster by sending huge number of requests from hundreds of clients
* Monitor the Pod and Cluster autoscaler. New Pods should get scheduled and new nodes should get added to the cluster
* Stop the test. Pods and nodes should get released and the cluster should come back to normal state

# Create AKS Cluster
While writing this article (Feb 10, 2019), the cluster autoscaler is in preview mode.
Open azure cloud shell <br>
* install the aks-preview Azure CLI extension using command : <br>
* <b>az extension add --name aks-preview</b> <br>
* Create AKS cluster <br>
* <b> az aks create --resource-group sidd-aks-poc-rg --name sidd-aks-poc-cluster2 --kubernetes-version 1.12.4 --node-count 3 --enable-vmss --enable-cluster-autoscaler --min-count 3 --max-count 10 </b> <br>

# Create Deployment & Load balancer
Following configuration files will be used:
* helloindia-deployment.yaml - Deployment for helloindia
* helloindia-service.yaml - Load balancer for helloindia service
* Create deployment :   <b>kubectl apply -f helloworld-deployment.yaml</b>
* Create service : <b>kubectl apply -f helloworld-service.yaml</b>

# Create Pod Autoscaler
* Create pod autoscaler. Min pods as 5 and Max as 40. Whenver CPU goes beyond 50%, try adding more pods
* Use this command :  <b>kubectl autoscale deployment helloindia --cpu-percent=50 --min=5 --max=40 </b>

# Verify Horizontal pod autoscaler, Pods And Nodes of the cluster
![autoscale-1](https://github.com/spatnaik77/azure-kubernetes-cluster-scalability/blob/master/diagrams/autoscale-1.png)





