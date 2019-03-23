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
* <b> az aks create --resource-group sidd-aks-poc-rg --name sidd-aks-poc-cluster2 --kubernetes-version 1.12.6 --node-count 3 --enable-vmss --enable-cluster-autoscaler --min-count 3 --max-count 10 </b> <br>
* <b> az aks get-credentials --resource-group sidd-aks-poc-rg --name sidd-aks-poc-cluster2 </b> <br>

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

# Start the Load test
You can use the cloud load tester available in Azure devops. Use the sidd-k8s-test.jmx for the test. This jmx script creates 25 threads so you can choose 10 agents to run the test which will simulate 250 concurrent users. select 30 mins as the duration of the test

# Monitor the auto scaling
Once the load test starts, periodically monitor the CPU usage of nodes, number of POD instances, number of node instances etc. You will see that pods and nodes are getting added gradually. See the below pictures:

![autoscale-3](https://github.com/spatnaik77/azure-kubernetes-cluster-scalability/blob/master/diagrams/autoscale-2.png)

After some time, now the cluster size is 10 ! <br>

![autoscale-3](https://github.com/spatnaik77/azure-kubernetes-cluster-scalability/blob/master/diagrams/autoscale-3.png)

# Stop the Load test
Once the load test is stopped, wait for atleast 15-30 mins to see that the nodes are removed from the cluster and the cluster comes back to initial state
![autoscale-1](https://github.com/spatnaik77/azure-kubernetes-cluster-scalability/blob/master/diagrams/autoscale-1.png)

# Performance test details
We started the test with 250 users. The initial cluster size was 3. As a result of autoscaling, more hardware was added and as a result, the response time decreased and the the throughput increased. See the below charts:

![report-charts](https://github.com/spatnaik77/azure-kubernetes-cluster-scalability/blob/master/diagrams/report-charts.png)

![report-summary](https://github.com/spatnaik77/azure-kubernetes-cluster-scalability/blob/master/diagrams/report-summary.png)

# References
https://docs.microsoft.com/en-gb/azure/aks/cluster-autoscaler .  <br>
https://docs.microsoft.com/en-gb/azure/aks/scale-cluster
