= Verdaccio setup as container at Google Cloud Kubernetes

Documentation how to setup up a private repo with verdaccio on Google Cloud Kubernetes. The repo is configured to only allow authenticated users to access, and of course publish, packages. It is assumed that you have installed `gcloud` and `kubectl` from the Google Cloud SDK. 

Make sure the current context of the is the project you want to install Verdaccio in.

```
gcloud config list
```

Update `gcloud` and install `kubectl` if missing:

```
gcloud components update
gcloud components install kubectl
```

Sign in 

```
gcloud auth login
```

To access the kubernetes dashboard you need to have a token to sign in. Get token and start proxy:

```
kubectl config view | grep -A10 "name: $(kubectl config current-context)" | awk '$1=="access-token:"{print $2}'
kubectl proxy -p 8888
```

Open your browser at http://localhost:8888/ui/ and select token and enter the token printed to shell. You should now see the kubernetes environment.

The installation of verdaccio is done using Helm, for the ease of it.


== Install Helm client on Mac

```
brew install kubernetes-helm
```

== Install tiller and permissions to kubernetes

Add permissions

```
kubectl create -f rbac.yaml 
```

Install Tiller in kubernetes

```
helm init --service-account tiller --upgrade
```

== Install Verdaccio with public IP

Get static IP for service

```
gcloud compute addresses create verdaccio-static-ip
```

Edit `values.yaml` and add the newly assigned IP. This file contains the values that will override the default settings. Check for example max users, in the current setting there is a limit of just one person being able to be added.

```
  loadBalancerIP: "<ENTER_PUBLIC_IP_HERE>"
```

Now for the actual installation:

helm install --name=myname -f values.yaml stable/verdaccio


Check installation using kubernetes dashboard at http://localhost:8888/ui/

You should soon be able to check your shiny new private repo at the assigned IP - http://<ASSIGNED_IP>:<PORT>


= Using Yarn with new private repo

When using yarn I couldn't get it working out of the box. I signed in with `yarn login` but when trying to add a new package with `yarn add mypackage` it always failed with `unregistered users are not allowed to access package mypackage".`

I had to wipe my `~/.npmrc`from all other stuff and added this line to make it work:

```
registry=http://username:password@domainname:4873/
```