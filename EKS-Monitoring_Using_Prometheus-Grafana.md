# Monitoring using Prometheus and Grafana

## pre-requisite
    
   1. EKS cluster active and Running
   2. Nodes must be up and Running
   3. Helm Installed on machine

## A. Installation and configuration of prometheus

We will use the helm to install Prometheus and Grafana

1. First create a namespace for prometheus in EKS cluster
    
        kubectl create namespace prometheus

2.  Add prometheus repository on cluster using helm
        
        helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

3. Install Prometehus using helm
      
        helm install prometheus prometheus-community/prometheus \
        --namespace prometheus \
        --set alertmanager.persistentVolume.storageClass="gp2" \
        --set server.persistentVolume.storageClass="gp2" \
        --set server.service.type=LoadBalancer

output :
![image](https://user-images.githubusercontent.com/101565681/220352015-81ebed13-bcd5-4ff9-a025-1ea5e2fb11cf.png)

Check Prometehus is 

    
