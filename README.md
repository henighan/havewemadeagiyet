# Have We Made AGI Yet?

This repos holds the resources for making website [havewemadeagiyet.com](http://www.havewemadeagiyet.com).
I shamelessly stole the idea from the creators of [hasthelargehadroncolliderdestroyedtheworldyet.com](http://www.hasthelargehadroncolliderdestroyedtheworldyet.com/). Note that while I decided to use kubernetes as a learning exercise, there are much simpler ways of deploying a static webpage like this. 

## Purchase Domain Name
I bought the domain name havewemadeagiyet.com through [google domains](https://domains.google/). It was not registered by anyone so it was pretty easy.

## Google Cloud
I made a google cloud account and followed the directions for making a project called `havewemadeagiyet` and a kubernetes cluster with the same name and default settings in zone us-west1-a


## The html
[index.html](index.html) holds the static webpage. To see it, you can just open it with your web-browser of choice using file->open

## Docker image
[Dockerfile](Dockerfile) holds the docker image definiton. Its a nginx (a popular webserver) docker image where I added the above html file as the static webpage to be served.

### Build the docker image:
```
cd havewemadeagi
docker build -t havewemadeagiyet .
```

### Tag the image for the google docker registry
```
docker tag havewemadeagiyet gcr.io/havewemadeagiyet/havewemadeagiyet
```

### Push the tagged image to the reigstry
```
docker push gcr.io/havewemadeagiyet/havewemadeagiyet
```
You will probably have to do some additional authentication stuff to make the above work, just follow the directions

## Create Static IP address
### Create
```
gcloud compute addresses create madeagi-ip --global
```
### Get IP address
```
gcloud compute addresses describe madeagi-ip --global
```
Put the IP address shown into the `loadBalancerIP` field of [webserver-service.yaml](webserver-service.yaml).

## Deploy Kubernetes Resources
In the google cloud web console, navigate to the kubernetes cluster and launch the "cloud shell". Once its up and running, do this to authenticate:
```
gcloud container clusters get-credentials havewemadeagiyet --zone us-west1-a
```
Now copy [webserver-deploy.yaml](./webserver-deploy.yaml) and [webserver-service.yaml](webserver-service.yaml) from this directory into the cloud shell. Now launch them as follows:
```
kubectl create -f ./webserver-deploy.yaml
kubectl create -f ./webserver-service.yaml
```
You should now have 3 replicas of the webserver pod running, which you can check with
```
kubectl get pods
```
You should also have a service running, which is showing its external IP as the one we put in the yaml file
```
kubectl get service
```

At this point, the site should be hosted publicly at the IP address we keep talking about. If you copy-paste that IP address into your browser, you should see the site.

## Point Domain Name to the Application
To link the purchased domain name havewemadeagiyet.com to this IP, I followed [these directions](https://cloud.google.com/dns/docs/quickstart#create_a_new_record). 
