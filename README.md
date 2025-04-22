# K8s deployment for vanilla DMP Roadmap.

### Build and Push Docker Image

Copy the Dockerfile to the root of the Roadmap project.
Then build and push the image to your registry.

docker build -t your-registry/dmproadmap:latest .
docker push your-registry/dmproadmap:latest


### Create a Secret for sensitive information

#### Delete old if exists
k get secrets                                     
  
  NAME                 TYPE     DATA   AGE
  dmproadmap-secrets   Opaque   3      8d

k delete secrets dmproadmap-secrets 
   
   secret "dmproadmap-secrets" deleted



kubectl create secret generic dmproadmap-secrets \
 --from-literal=DB_USERNAME=postgres \
 --from-literal=DB_PASSWORD=roadmap \
 --from-literal=RAILS_MASTER_KEY=c46dc1c1e9c61df391c13f3af8b90d68

See the secret
k get secrets                                 

NAME                 TYPE     DATA   AGE
dmproadmap-secrets   Opaque   3      6s


### Deploy to Kubernetes

kubectl apply -f configmap.yml

kubectl apply -f postgres.yml

kubectl apply -f dmproadmap.yml

kubectl apply -f ingress.yml

### Initialize the Database and do rails db:migrate

kubectl exec -it $(kubectl get pods -l app=dmproadmap -o jsonpath="{.items[0].metadata.name}") -- bundle exec rails db:setup

kubectl exec -it $(kubectl get pods -l app=dmproadmap -o jsonpath="{.items[0].metadata.name}") -- bundle exec rails external_api:load_spdx_licenses

kubectl exec -it $(kubectl get pods -l app=dmproadmap -o jsonpath="{.items[0].metadata.name}") -- bundle exec rails external_api:load_rdamsc_standards

kubectl exec -it $(kubectl get pods -l app=dmproadmap -o jsonpath="{.items[0].metadata.name}") -- bundle exec rails external_api:add_field_of_science_to_research_domains

kubectl exec -it $(kubectl get pods -l app=dmproadmap -o jsonpath="{.items[0].metadata.name}") -- bundle exec rails db:migrate

### Precompile Assests
kubectl exec -it $(kubectl get pods -l app=dmproadmap -o jsonpath="{.items[0].metadata.name}") -- bash -c "DISABLE_SPRING=1 bundle exec rails assets:precompile"


### Troubleshooting if  http://dmproadmap.local not working T
#### Test direct access to the pod
Bypass the Ingress to test if the application is working:

kubectl port-forward $(kubectl get pods -l app=dmproadmap -o jsonpath="{.items[0].metadata.name}") 3000:3000

Then try accessing http://localhost:3000 in your browser.