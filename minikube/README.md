# Confluent Platform 5.5 Operator Demo (Alec's local minikube setup)
 
## Setup Kubernetes

```
minikube start --cpus=4 --memory=8G

# should match k8s version
kubectl version --short

minikube dashboard &

# In a separate terminal window, create a minikube tunnel for local DNS
sudo minikube tunnel
```

## Download the Operator bundle from Confluent website
```
wget https://platform-ops-bin.s3-us-west-1.amazonaws.com/operator/confluent-operator-5.5.0.tar.gz
tar -xvf confluent-operator-5.5.0.tar.gz
cd confluent-operator/
```

## Extend Kubernetes with first class CP primitives

```
kubectl apply --filename ./resources/crds/
```

## Create a Kubernetes namespace to install Operator and CP

```
kubectl create namespace confluent
kubectl config set-context --current --namespace confluent
```

## Configure and deploy Operator and CP

```
cp Downloads/confluent-operator/helm/providers/private.yaml /tmp/values.yaml

vim /tmp/values.yaml # make changes

vimdiff /tmp/values.yaml Downloads/confluent-operator/helm/providers/private.yaml

#all in one shot
helm install confluent-platform \
  --values=/tmp/values.yaml \
  --namespace=confluent \
  --set operator.enabled=true \
  --set zookeeper.enabled=true \
  --set kafka.enabled=true \
  --set controlcenter.enabled=true \
```

## Wait for CP to start up

```
# Watch the Kubernetes dashboard
```

## Validate the installation with Control Center

```
echo \
  $(kubectl get service controlcenter-bootstrap-lb \
      --output=jsonpath={'.status.loadBalancer.ingress[0].ip'} \
      --namespace=confluent) \
  controlcenter.confluent.platform.55.demo | sudo tee -a /etc/hosts

open http://controlcenter.confluent.platform.55.demo # admin / Developer1

# Go to cluster -> topics -> /_confluent-metrics/message-viewer
```

## Confirm the use of Red Hat images

```
kubectl exec controlcenter-0 -nconfluent -- cat /etc/redhat-release
```

## Clean up

```
# Un-hack /etc/hosts

minikube delete
```
