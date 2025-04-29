# K8s deployment for vanilla DMP Roadmap.

### Build and Push Docker Image

Copy the Dockerfile-dmproadmap to the root of the Roadmap project.
Then build and push the image to your registry.

docker build -t your-registry/dmproadmap:latest .
docker push your-registry/dmproadmap:latest


oadmamp
git clone https://github.com/DMPRoadmap/roadmap.git
cd roadmap
git co development
git pull




# Create a Kubernetes cluster with Minikube

### Install Minikube on your local machine.
https://minikube.sigs.k8s.io/docs/start/?arch=%2Flinux%2Fx86-64%2Fstable%2Fbinary+download


### Start a cluster using the virtualbox driver:

> minikube start --driver=virtualbox

To make virtualbox the default driver:

> minikube config set driver virtualbox

### Enable the Ingress addon in Minikube

minikube addons enable ingress

### Setting up a domain with Minikube

Get your Minikube IP

> minikube ip

My output: 192.168.59.100

### Update your hosts file to map a domain to this IP

> sudo vi /etc/hosts

Add the following line to the end of the file:
192.168.59.100    dmproadmap.local


### Create a Secret for sensitive information

#### Delete old if exists
> kubectl get secrets                                     
  
  NAME                 TYPE     DATA   AGE
  dmproadmap-secrets   Opaque   3      8d

> kubectl delete secrets dmproadmap-secrets 
   
   secret "dmproadmap-secrets" deleted



> kubectl create secret generic dmproadmap-secrets \
 --from-literal=DB_USERNAME=postgres \
 --from-literal=DB_PASSWORD=roadmap \
 --from-literal=RAILS_MASTER_KEY=c46dc1c1e9c61df391c13f3af8b90d68

See the secret
 > k get secrets                                 

NAME                 TYPE     DATA   AGE
dmproadmap-secrets   Opaque   3      6s


### Deploy to Kubernetes

> kubectl apply -f configmap.yml

> kubectl apply -f postgres.yml

> kubectl apply -f dmproadmap.yml

> kubectl apply -f ingress.yml

### Initialize the Database and do rails db:migrate

kubectl exec -it $(kubectl get pods -l app=dmproadmap -o jsonpath="{.items[0].metadata.name}") -- bundle exec rails db:setup

kubectl exec -it $(kubectl get pods -l app=dmproadmap -o jsonpath="{.items[0].metadata.name}") -- bundle exec rails external_api:load_spdx_licenses

kubectl exec -it $(kubectl get pods -l app=dmproadmap -o jsonpath="{.items[0].metadata.name}") -- bundle exec rails external_api:load_rdamsc_standards

kubectl exec -it $(kubectl get pods -l app=dmproadmap -o jsonpath="{.items[0].metadata.name}") -- bundle exec rails external_api:add_field_of_science_to_research_domains

kubectl exec -it $(kubectl get pods -l app=dmproadmap -o jsonpath="{.items[0].metadata.name}") -- bundle exec rails db:migrate

### Troubleshooting if  http://dmproadmap.local not working

## Test direct access to the pod

Bypass the Ingress to test if the application is working:

> kubectl port-forward $(kubectl get pods -l app=dmproadmap -o jsonpath="{.items[0].metadata.name}") 3000:3000

Then try accessing http://localhost:3000 in your browser.


#### Test Users

The test user accounts below all have the password: password123

Demo Super Administrator = super_admin@example.com

Demo Funder Administrator = funder_admin@example.com

Demo Organisation Administrator = org_admin@example.com

Demo Organisation User = org_user@example.com