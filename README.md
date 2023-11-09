# K8s-services
How we use k8s-services to expose the application tot he outside world as well as within our k8s cluster and within our organization. <br/>

First we create an image using [Dockerfile](https://github.com/warlock601/K8s-services/blob/c88d7e410bfdb916fc008bec988b7b76dfe28e7b/Dockerfile) 
```bash
  docker build -t vivekraj/python-web-app
```
We build an image with name "vivekraj/python-web-app" <br/>
Then we do the deployment. And in our Deploymet, we will use the same image-name.

