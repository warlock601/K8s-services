# K8s-services
How we use k8s-services to expose the application tot he outside world as well as within our k8s cluster and within our organization. <br/>

First we create an image using [Dockerfile](https://github.com/warlock601/K8s-services/blob/c88d7e410bfdb916fc008bec988b7b76dfe28e7b/Dockerfile) 
```bash
  docker build -t vivekraj/python-web-app
```
We build an image with name "vivekraj/python-web-app" <br/>
Then we do the deployment. We can check out the deployment file from Official [Documentation](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/) from kubernetes. <br/>
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
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

```

Then we edit the Deployment file as per our application (tags,labels) and also use the image that we build by specifying the image-name in our deployment.<br/>
Create the deployment using:
```bash
  kubectl apply -f deployment.yml
```
A Deployment will be created which will create Replica sets which will further create Pods. The no. of replicas that we specified, those many pods will be created.<br/>


In Kubernetes, pods being shut down or deleted is normal and when a new replica is created, a new IP address is allocated to the new pod as because k8s does a dynamic allocations of IPs. That's why we need Service discovery mechanism. It uses the concept of Selectors and Labels instead of IP addresses so that when a Pod comes up with a new IP, service won't be looking for an IP instead it will be looking for Selectors and Labels (as the  service will route traffic based on selectors and labels. the pods that will replace a new pod will have the same label). <br/>
First we'll get IPs of the pods running.
```bash
  kubectl gert pods -o wide
```
![Screenshot 2023-11-11 164230](https://github.com/warlock601/K8s-services/assets/32487715/a60c111c-824e-4ac8-8db6-78e6fc396ba3)
And then SSH into the cluster.
```bash
   minikube ssh                      
```
Use the IP & login into the cluster and try to access some traffic using the context route:
```bash
   curl -L hhtp://10.244.0.12:8000/demo                      
```
Where 8000 is the Port no. & "demo" is the context route that is defined in the Django application. Here we are able to access the context route as we accessed it from inside the cluster.

![Screenshot 2023-11-12 050258](https://github.com/warlock601/K8s-services/assets/32487715/15c1fd9b-7005-4580-bed7-9dcdae588d0f)

If we try to access the Application & try to access the trafiic from outside the Cluster, we won't be able to do so. We will not get any response because by default a Pod willl only have the Cluster IP addresss attached to it. Basically if it is a cluster network, we have to access it using the cluster itself.

![Screenshot 2023-11-12 051005](https://github.com/warlock601/K8s-services/assets/32487715/834010bc-6d3e-4de0-9a3a-90b27f84bf9d)

Now, the users who want to access this application can be inside or even outside the organization.
- For users inside our organization but outside our cluster, we can expose our application on the k8s worker node IP. **NodePort** mode will be used here (application will be exposed on the Node IP address).
- For users outside our organization, they don't even have access to even our organization so we need public IPs for this app so that anyone in the world can access this app. **Load Balancer** mode will be used here. <br/>
<br/>
To use these 2 modes, we need K8s service. In our case, the node IP is the minikube IP address because we're using minikube. Services will only bother bother baout the labels and selectors.
Note: For Service label, always copy the label from Deployments-Template as it might have different selectors and labels for Deployment.
<br/>
<br/>
![Screenshot 2023-11-12 053944](https://github.com/warlock601/K8s-services/assets/32487715/8724201f-b18e-43cd-a370-3e249185df34)

We created serivce using [service.yml](https://github.com/warlock601/K8s-services/blob/4233563797769f5f025df8cc4f5a22c1965d77be/service.yml) file. We check the running services.
```bash
   kubectl get svc                    
```
Because we've created the service using NodePort mode, there is a port-mapping that is done. Cluster Ip is mapped with Node IP and port 30070.

![Screenshot 2023-11-12 062353](https://github.com/warlock601/K8s-services/assets/32487715/5e184cfe-5302-494c-9885-c9367e17265d)

Now when we try to access the trafiic using Node IP (from outside the cluster), we're able to, as the service maps Node IP to the cluster IP given that we'll be using minikube IP along with node port no.
Here we've mapped cluster port to the node port-30070. And we can even access the traffic through a browser.

![Screenshot 2023-11-12 070739](https://github.com/warlock601/K8s-services/assets/32487715/c9954361-e851-4897-8c09-24e8d1f24661)

But if we take the same URL & try to access the traffic from outside, it will not work as we have not exposed our application to the outside world. So, to make that possible, we change the type to LoadBalancer in service.yml file.
<br/>
<br/>
![Screenshot 2023-11-12 081959](https://github.com/warlock601/K8s-services/assets/32487715/730d7f61-488d-4c99-8fa6-8a3423f85db9)

![Screenshot 2023-11-12 082026](https://github.com/warlock601/K8s-services/assets/32487715/f16a984c-accd-4243-9ed0-7f53305b09c8)

We're using Minikube here so here we won't be able to see any difference as it will work only inside a Cloud Provider as LoadBalancer type is only applicable with a cloud provider.
Here we see that the External IP is pending & if instead of minikube we've used any cloud provider, it would've been assigned an external IP. CCM of k8s will be genrating this external IP.


![Screenshot 2023-11-12 082738](https://github.com/warlock601/K8s-services/assets/32487715/7e91f6d5-4fe4-4fe4-bd7e-394470f6cc3a)

We can use MetalLB to assign a public IP. <br/>
We can use KubeShark to Monitor the traffic for k8s, as it can stream live traffic (we can also use it to check the load balancing done to the pods running behind a k8s service).
