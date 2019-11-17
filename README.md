this application will create minikube cluster with 2 ngnix containers ,with loging solution that contain Elastic search and kibana and the logs will move from fluentd .

1. install minikube cluster :

Run this :

curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-darwin-amd64 \\n  && chmod +x minikube

sudo mv minikube /usr/local/bin

minikube config set memory 4096

minikube start

2. create 2 ngnix dummy containers to create logs 

kubectl run nginxapp --image=nginx:latest --port=80\ndeployment "mynginxapp" create
kubectl run nginxapp1 --image=nginx:latest --port=80\ndeployment "mynginxapp" created

3. install Elastic 

kubectl apply -f https://download.elastic.co/downloads/eck/1.0.0-beta1/all-in-one.yaml

cat <<EOF | kubectl apply -f -
apiVersion: elasticsearch.k8s.elastic.co/v1beta1
kind: Elasticsearch
metadata:
  name: quickstart
spec:
  version: 7.4.2
  nodeSets:
  - name: default
    count: 1
    config:
      node.master: true
      node.data: true
      node.ingest: true
      node.store.allow_mmap: false
EOF

to get elastic password with the default user "elastic" run :

PASSWORD=$(kubectl get secret quickstart-es-elastic-user -o=jsonpath='{.data.elastic}' | base64 --decode)

To check elastic is working :

From your local workstation, use the following command in a separate terminal:

kubectl port-forward service/quickstart-es-http 9200

Then request localhost:

curl -u "elastic:$PASSWORD" -k "https://localhost:9200"

4 install Kibana 

cat <<EOF | kubectl apply -f -
apiVersion: kibana.k8s.elastic.co/v1beta1
kind: Kibana
metadata:
  name: quickstart
spec:
  version: 7.4.2
  count: 1
  elasticsearchRef:
    name: quickstart
EOF

Access Kibana:

A ClusterIP Service is automatically created for Kibana:

kubectl get service quickstart-kb-http


Use kubectl port-forward to access Kibana from your local workstation:

kubectl port-forward service/quickstart-kb-http 5601

https://localhost:5601

user :elastic 
pass: kubectl get secret quickstart-es-elastic-user -o=jsonpath='{.data.elastic}' | base64 --decode; echo


5. fluentd deploy :

Apply this Yaml -  fluentd-daemonset-elasticsearch-rbac.yaml   

Aplly the configmap - fluentd-configmap-elasticsearch-rbac.yaml

6. access kibana add the index_name fluend-ngnix     to see the logs in Kibana .


