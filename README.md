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


