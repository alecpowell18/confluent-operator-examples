# Confluent Platform 5.5 Operator Demo on Amazon EKS

### *STEPS (k8s v1.16.8) - Tested by Alec 05-28-20*

## Create cluster
```
eksctl create cluster --name=alec-eks --tags="Owner=apowell" --version=1.16 --region=us-west-2 --zones=us-west-2a,us-west-2b,us-west-2c --nodegroup-name=alec-workers1 --node-type=m5.xlarge --nodes=6 --nodes-min=4 --nodes-max=6 --ssh-access --ssh-public-key=alec-west-2 --managed --external-dns-access
```
Output looks something like:
```
[ℹ]  eksctl version 0.19.0
[ℹ]  using region us-west-2
[ℹ]  subnets for us-west-2a - public:192.168.0.0/19 private:192.168.96.0/19
[ℹ]  subnets for us-west-2b - public:192.168.32.0/19 private:192.168.128.0/19
[ℹ]  subnets for us-west-2c - public:192.168.64.0/19 private:192.168.160.0/19
[ℹ]  using EC2 key pair %!!(MISSING)q(*string=<nil>)
[ℹ]  using Kubernetes version 1.16
[ℹ]  creating EKS cluster "alec-eks" in "us-west-2" region with managed nodes
[ℹ]  will create 2 separate CloudFormation stacks for cluster itself and the initial managed nodegroup
[ℹ]  if you encounter any issues, check CloudFormation console or try 'eksctl utils describe-stacks --region=us-west-2 --cluster=alec-eks'
[ℹ]  CloudWatch logging will not be enabled for cluster "alec-eks" in "us-west-2"
[ℹ]  you can enable it with 'eksctl utils update-cluster-logging --region=us-west-2 --cluster=alec-eks'
[ℹ]  Kubernetes API endpoint access will use default of {publicAccess=true, privateAccess=false} for cluster "alec-eks" in "us-west-2"
[ℹ]  2 sequential tasks: { create cluster control plane "alec-eks", create managed nodegroup "alec-workers1" }
[ℹ]  building cluster stack "eksctl-alec-eks-cluster"
[ℹ]  deploying stack "eksctl-alec-eks-cluster"
[ℹ]  building managed nodegroup stack "eksctl-alec-eks-nodegroup-alec-workers1"
[ℹ]  deploying stack "eksctl-alec-eks-nodegroup-alec-workers1"
[ℹ]  waiting for the control plane availability...
[✔]  saved kubeconfig as "/Users/apowell/.kube/config"
[ℹ]  no tasks
[✔]  all EKS cluster resources for "alec-eks" have been created
[ℹ]  nodegroup "alec-workers1" has 6 node(s)
[ℹ]  node "ip-192-168-12-138.us-west-2.compute.internal" is ready
[ℹ]  node "ip-192-168-28-202.us-west-2.compute.internal" is ready
[ℹ]  node "ip-192-168-43-203.us-west-2.compute.internal" is ready
[ℹ]  node "ip-192-168-60-140.us-west-2.compute.internal" is ready
[ℹ]  node "ip-192-168-90-80.us-west-2.compute.internal" is ready
[ℹ]  node "ip-192-168-91-78.us-west-2.compute.internal" is ready
[ℹ]  waiting for at least 4 node(s) to become ready in "alec-workers1"
[ℹ]  nodegroup "alec-workers1" has 6 node(s)
[ℹ]  node "ip-192-168-12-138.us-west-2.compute.internal" is ready
[ℹ]  node "ip-192-168-28-202.us-west-2.compute.internal" is ready
[ℹ]  node "ip-192-168-43-203.us-west-2.compute.internal" is ready
[ℹ]  node "ip-192-168-60-140.us-west-2.compute.internal" is ready
[ℹ]  node "ip-192-168-90-80.us-west-2.compute.internal" is ready
[ℹ]  node "ip-192-168-91-78.us-west-2.compute.internal" is ready
[ℹ]  kubectl command should work with "/Users/apowell/.kube/config", try 'kubectl get nodes'
[✔]  EKS cluster "alec-eks" in "us-west-2" region is ready
```

## [Optional] Deploy Kubernetes Dashboard
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml
kubectl proxy --port=8080 --address=0.0.0.0 --disable-filter=true &


#paste in browser: 

open http://localhost:8080/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login
# in a new terminal tab, get a token to login
aws eks get-token --cluster-name alec-eks | jq -r '.status.token'
# paste token into the browser
```
## Confluent Operator setup
Download Operator
```
wget https://platform-ops-bin.s3-us-west-1.amazonaws.com/operator/confluent-operator-5.5.0.tar.gz
tar -xvf confluent-operator-5.5.0.tar.gz
cd confluent-operator/
```

Create new values.yaml file for EKS
```
cp helm/providers/aws.yaml alec-eks-values.yaml
vim alec-eks-values.yaml  # update region and zones, enable&configure LB for external access
```

```

kubectl create namespace confluent
kubectl config set-context --current --namespace confluent

#NOTE: make sure you are in the top-level confluent-operator/ directory, or otherwise point paths correctly with below commands.
#helm install <component> <path-to-helm-charts> --values <path-to-values.yaml> --namespace <your-namespace> --set <component>.enabled=true

helm install operator ./helm/confluent-operator --values alec-eks-values.yaml --namespace confluent --set operator.enabled=true

kubectl get pods #get podname from here
kubectl get crd | grep confluent
kubectl logs cc-operator-xxxxxx |less -S  ## update podname accordingly, check that operator is running and healthy

helm install zookeeper ./helm/confluent-operator --values alec-eks-values.yaml --namespace confluent --set zookeeper.enabled=true
helm install kafka ./helm/confluent-operator --values alec-eks-values.yaml --namespace confluent --set kafka.enabled=true

helm install controlcenter ./helm/confluent-operator --values alec-eks-values.yaml --namespace confluent --set controlcenter.enabled=true
```

To expose C3 locally:
```
kubectl -n confluent port-forward controlcenter-0 12345:9021
open http://localhost:12345
```

## Connect to Kafka
```
#get info (including internal bootstrap server)
kubectl get kafka -n confluent -oyaml
# ssh to a pod:
kubectl -n c-operator exec -it kafka-0 bash
# create properties file in the pod:
cat << EOF > kafka.properties
bootstrap.servers=kafka:9071
sasl.jaas.config=org.apache.kafka.common.security.plain.PlainLoginModule required username="test" password="test123";
sasl.mechanism=PLAIN
security.protocol=SASL_PLAINTEXT
EOF
# query bootstrap server
kafka-broker-api-versions --command-config kafka.properties --bootstrap-server kafka:9071
#exit the server
exit
```

## ksqlDB
```
#If load balancer was enabled=true for ksqlDB, you can connect from the outside at port 80
ksql http://<your-lb>.us-west-2.elb.amazonaws.com:80
SET 'auto.offset.reset'='earliest';
```

## Monitoring
```
#install prometheus, grafana
helm repo add stable https://kubernetes-charts.storage.googleapis.com
helm install demo-test stable/prometheus \
     --set alertmanager.persistentVolume.enabled=false \
     --set server.persistentVolume.enabled=false \
     --namespace confluent
helm install grafana stable/grafana --namespace confluent
kubectl port-forward grafana-5bf4d98fff-ss9l6 3000
#retrieve admin password
kubectl get secret --namespace confluent grafana -o jsonpath="{.data.admin-password}" | base64 --decode
#login, configure Prometheus data source printing to prometheus-server LB or pod clusterIP
kubectl get svc demo-test-prometheus-server
#import dashboard JSON file from local
```
 
## Delete cluster
```
eksctl delete cluster --name alec-eks
```
