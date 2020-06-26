# Confluent Platform 5.5 Operator Demo on Azure Kubernetes Service

### *STEPS (k8s v1.16.8) - Tested by @Alec Powell 06.16.20*

## Deploy AKS cluster
See which versions of k8s are available to you:
```
az aks get-versions --location eastus --output table
```
Spin up AKS cluster (takes few mins):

I created an ssh key (azure_key) ahead of time (ssh-keygen) specifically for AKS such that I could log into any of the VMs if needed. I didn't end up using it.
```
az aks create \
    --resource-group alec-demo \
    --name alecAKSCluster \
    --kubernetes-version 1.16.8 \
    --node-count 3 \
    --load-balancer-sku basic \
    --disable-rbac \
    --ssh-key-value ~/.ssh/azure_key.pub
```
You might get an error like this, disregard and try again -->
> "Operation failed with status: 'Bad Request'. Details: The credentials in ServicePrincipalProfile were invalid. Please see https://aka.ms/aks-sp-help for more details. (Details: token contains an invalid number of segments)"


When it's done, you will see:
```
{- Finished ..
  "aadProfile": null,
  "addonProfiles": null,
  "agentPoolProfiles": [
    {
      "availabilityZones": null,
      "count": 3,
      "enableAutoScaling": null,
      "enableNodePublicIp": false,
      "maxCount": null,
      "maxPods": 110,
      "minCount": null,
      "mode": "System",
      "name": "nodepool1",
      "nodeLabels": {},
      "nodeTaints": null,
      "orchestratorVersion": "1.16.8",
      "osDiskSizeGb": 100,
      "osType": "Linux",
      "provisioningState": "Succeeded",
      "scaleSetEvictionPolicy": null,
      "scaleSetPriority": null,
      "spotMaxPrice": null,
      "tags": null,
      "type": "VirtualMachineScaleSets",
      "vmSize": "Standard_D2s_v3",
      "vnetSubnetId": null
    }
  ],
  "apiServerAccessProfile": null,
  "autoScalerProfile": null,
  "diskEncryptionSetId": null,
  "dnsPrefix": "alecAKSClu-alec-demo-54ff81",
  "enablePodSecurityPolicy": null,
  "enableRbac": true,
  "fqdn": "alecaksclu-alec-demo-54ff81-39505dde.hcp.eastus.azmk8s.io",
  "id": "/subscriptions/54ff81a0-e7f6-4919-9053-4cdd1c5f5ae1/resourcegroups/alec-demo/providers/Microsoft.ContainerService/managedClusters/alecAKSCluster",
  "identity": null,
  "identityProfile": null,
  "kubernetesVersion": "1.16.8",
  "linuxProfile": {
    "adminUsername": "azureuser",
    "ssh": {
      "publicKeys": [
        {
          "keyData": "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDA+G/C7Fk1W2esdHWz34PYOjTk4CQ0YRsFfk+9rw8BsACq3Q8g0YhOA16VmwgLwDb+aoCKhs0G0VYeOOU/CHxe01ATPUtwqrkp94ah4zYO+aF+42cKwrGCFRG/etDvUl1NykjxnPQvIKco/T/HBNCelaD8E1ncYEQ9uw/+8I7+t5O+9cyGQUrq2KlB5cYtA4KoeHYoEhPYw6rgo9syjRiXhl6o7EV0ZzXQlhxpp4NQt7TrqfnhcrpyxUPvAl3weVM44aDSN7HnRnJ2adawK2hv+D9xcy62TungQFfOw2XeJSuz8CgByNLm6DOSifRzIilg5e5jbpT4RGhFfoyvYfBSHSOM100fq3P120zMEMoDxtHcRaUFbrfNv3OxpxXRYdfte7HjVvV1CJAG5rfoSW7Wy4sdYplBlDZQiw5f59f4zwnuPVhn9vVGpbMq/7vIFx+heFAo8qOZ9oduPVrHe1Bp+cAH/bj54OjgPvc46hHN5pjZN9SRtcbt1S/TyAFmDHqtgzNM44g6VW1ucS41hRgUzMgLC7o3fIJJcSXnitSoWNDyGW7wZlLSK2UKjNDtydLGmzQmUJxvDWJSGPHPSQ13Y24KHq3bOzRliqKEji3XKivfwZPveWZJ2qpW86yBJCxXA0JINu3T/LaB9nB9Vevif7vZSAcN1P7LLzI6vkdhqw== azureuser@linuxvm\n"
        }
      ]
    }
  },
  "location": "eastus",
  "maxAgentPools": 10,
  "name": "alecAKSCluster",
  "networkProfile": {
    "dnsServiceIp": "10.0.0.10",
    "dockerBridgeCidr": "172.17.0.1/16",
    "loadBalancerProfile": null,
    "loadBalancerSku": "Basic",
    "networkMode": null,
    "networkPlugin": "kubenet",
    "networkPolicy": null,
    "outboundType": "loadBalancer",
    "podCidr": "10.244.0.0/16",
    "serviceCidr": "10.0.0.0/16"
  },
  "nodeResourceGroup": "MC_alec-demo_alecAKSCluster_eastus",
  "privateFqdn": null,
  "provisioningState": "Succeeded",
  "resourceGroup": "alec-demo",
  "servicePrincipalProfile": {
    "clientId": "33a3fcd4-bc28-47da-b4be-851b592f455f",
    "secret": null
  },
  "sku": {
    "name": "Basic",
    "tier": "Free"
  },
  "tags": null,
  "type": "Microsoft.ContainerService/ManagedClusters",
  "windowsProfile": null
}

```
Get your kubectl local pointed to your AKS cluster 
```
az aks get-credentials --resource-group alec-demo --name alecAKSCluster
Merged "alecAKSCluster" as current context in /Users/apowell/.kube/config
```
To see the VMs:
```
kubectl get nodes -owide
```
Deploy the k8s dashboard (optional)
```
az aks browse --resource-group alec-demo --name alecAKSCluster
kubectl create clusterrolebinding kubernetes-dashboard --clusterrole=cluster-admin --serviceaccount=kube-system:kubernetes-dashboard
open http://127.0.0.1:8001
```

## Confluent Operator setup
Download Operator
```
wget https://platform-ops-bin.s3-us-west-1.amazonaws.com/operator/confluent-operator-5.5.0.tar.gz
tar -xvf confluent-operator-5.5.0.tar.gz
cd confluent-operator/
```

Create new values.yaml file for AKS
```
cp helm/providers/azure.yaml alec-aks-values.yaml
vim alec-aks-values.yaml  # update region and zones, enable&configure LB for external access
```

Create a namespace, deploy the operator, and install Zookeeper/Kafka/C3
```
kubectl create namespace confluent
kubectl config set-context --current --namespace confluent

#NOTE: make sure you are in the top-level confluent-operator/ directory, or otherwise point paths correctly with below commands.
#helm install <component> <path-to-helm-charts> --values <path-to-values.yaml> --namespace <your-namespace> --set <component>.enabled=true

helm install operator ./helm/confluent-operator --values alec-aks-values.yaml --namespace confluent --set operator.enabled=true

kubectl get pods #get podname from here
kubectl get crd | grep confluent
kubectl logs cc-operator-xxxxxx |less -S  ## update podname accordingly, check logs to make sure Operator is running properly

helm install zookeeper ./helm/confluent-operator --values alec-aks-values.yaml --namespace confluent --set zookeeper.enabled=true
helm install kafka ./helm/confluent-operator --values alec-aks-values.yaml --namespace confluent --set kafka.enabled=true
helm install controlcenter ./helm/confluent-operator --values alec-aks-values.yaml --namespace confluent --set controlcenter.enabled=true
```

Expose C3 locally
```
kubectl -n confluent port-forward controlcenter-0 12345:9021
open http://localhost:12345 # admin / Developer1
```

## Configure DNS

### Using Azure DNS (optional)
The output from installing Kakfa will look like this:
```
External:

          - Bootstrap LB: kafka.apow-test.dev

          - Pod name: kafka-0
            DNS endpoints: b0.apow-test.dev
            AZURE LB Endpoint: kubectl -n confluent describe svc kafka-0-lb

          - Pod name: kafka-1
            DNS endpoints: b1.apow-test.dev
            AZURE LB Endpoint: kubectl -n confluent describe svc kafka-1-lb

          - Pod name: kafka-2
            DNS endpoints: b2.apow-test.dev
            AZURE LB Endpoint: kubectl -n confluent describe svc kafka-2-lb

        ***You can get all above information on the status field by running the below command**

        kubectl -n confluent get kafka kafka  -oyaml

            Create the DOMAIN: apow-test.dev and update LB endpoint for each DNS endpoint for each broker as necessary.
```

Run the below CLI commands to programatially create DNS records in Azure:
```
az network dns zone create -g alec-demo -n apow-test.dev
az network dns record-set a add-record -g alec-demo -z apow-test.dev -n kafka -a 40.88.7.203
az network dns record-set a add-record -g alec-demo -z apow-test.dev -n b0 -a 52.186.139.40
az network dns record-set a add-record -g alec-demo -z apow-test.dev -n b1 -a 52.152.221.205
az network dns record-set a add-record -g alec-demo -z apow-test.dev -n b2 -a 52.142.15.108

```

### Using local /etc/hosts
Or, if you wish to just hack your /etc/hosts file to bypass DNS creation, you can do so (only for dev/test purposes).

Your IPs will be different as well as FQDN (which is what you put in loadBalancer.domain in values.yaml)

```
# add the below lines to /etc/hosts file
52.186.139.40 b0.apow-test.dev b0
52.152.221.205 b1.apow-test.dev b1
52.142.15.108 b2.apow-test.dev b2
40.88.7.203 kafka.apow-test.dev kafka
```

## Testing

### Create test topic
Create file client-aks.properties
```
kafka-topics --bootstrap-server 40.88.7.203:9092 --command-config client-aks.properties --create --topic test --partitions 1 --replication-factor 1
```

### Produce to topic from within k8s pod 
```
# Get info (including internal bootstrap server)
kubectl get kafka -n confluent -oyaml
# ssh into a broker pod:
kubectl -n confluent exec -it kafka-0 bash
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


## Cleanup
```
helm delete controlcenter
helm delete kafka
helm delete zookeeper
helm delete operator
az dns delete --resource-group alec-demo --name alecDNSZone
az aks delete --resource-group alec-demo --name alecAKSCluster
az group delete --name alec-demo
```
