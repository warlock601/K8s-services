# K8s-services
How we use k8s-services to expose the application tot he outside world as well as within our k8s cluster and within our organization. <br/>

First we create an image using [Dockerfile]() 
```bash
  docker build -t vivekraj/python-web-app
```
We build an image with name "vivekraj/python-web-app" <br/>
Then we do the deployment. And in our Deploymet, we will use the same image-name.

