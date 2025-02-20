# EKS Cluster Installation

## Prerequisites
- Update `eksctl` to the latest version:
  ```bash
  eksctl version
  curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
  sudo mv /tmp/eksctl /usr/local/bin
  ```

## Cluster Configuration
- Set the following variables:
  ```bash
  cluster_name=crm-prod-apsoutheast1a
  region_code=ap-southeast-1
  kube_version=1.30 # you can change this with the latest version
  ```

## Create Cluster
- **Initial Cluster Creation**:
  ```bash
  eksctl create cluster --name $cluster_name --region $region_code --version $kube_version --without-nodegroup --node-private-networking
  ```

- **Custom Config Cluster Creation**:
  ```bash
  eksctl create cluster --name $cluster_name --region $region_code --version $kube_version --vpc-private-subnets subnet-0d440336f7d4c4930,subnet-0e49933c68c4f809e,subnet-0c9321758150e24a1 --without-nodegroup
  ```

## Node Group Configuration
- **Create Nodegroup without Launch Template**:
  ```bash
  eksctl create nodegroup \
    --cluster crm-production \
    --region ap-southeast-1 \
    --name worker-ng \
    --node-ami-family AmazonLinux2 \
    --node-type t3.medium \
    --nodes 3 \
    --nodes-min 0 \
    --nodes-max 7 \
    --node-volume-size 40 \
    --subnet-ids subnet-0dd08c4ab055062f4,subnet-0cd5cf572c209f703,subnet-08d0d231e13f37ba2 \
    --node-private-networking
  ```

- **Single Node Group Subnet**:
  ```bash
  eksctl create nodegroup \
    --cluster crm-production \
    --region ap-southeast-1 \
    --name ng-1-amd64 \
    --node-ami-family AmazonLinux2 \
    --node-type t3.medium \
    --nodes 3 \
    --nodes-min 0 \
    --nodes-max 7 \
    --node-volume-size 40 \
    --subnet-ids subnet-0dd08c4ab055062f4 \
    --node-private-networking \
    --managed
  ```

- **ARM64 Node Group**:
  ```bash
  eksctl create nodegroup \
    --cluster crm-production \
    --region ap-southeast-1 \
    --name worker-prod-arm64 \
    --node-ami-family AmazonLinux2 \
    --node-type m6g.large \
    --nodes 2 \
    --nodes-min 0 \
    --nodes-max 6 \
    --node-volume-size 40 \
    --subnet-ids subnet-0dd08c4ab055062f4,subnet-0cd5cf572c209f703,subnet-08d0d231e13f37ba2 \
    --node-private-networking \
    --managed
  ```

- **Update Nodegroup**:
  ```bash
  eksctl update nodegroup --cluster=crm-production --name=worker --node-volume-size=40
  ```

## Load Balancer Configuration
- **Nginx Ingress Controller**:
  - [Guide Link](https://aws.amazon.com/blogs/containers/exposing-kubernetes-applications-part-3-nginx-ingress-controller/)
  - [Guide Link (Indonesian)](https://aws.amazon.com/id/blogs/indonesia/mengekspos-aplikasi-kubernetes-bagian-3-nginx-ingress-controller/)

- **ALB**:
  - Create IAM Policy:
    ```bash
    aws iam create-policy \
      --policy-name AWSLoadBalancerControllerIAMPolicy \
      --policy-document file://iam_policy.json
    ```

  - Associate IAM OIDC Provider:
    ```bash
    eksctl utils associate-iam-oidc-provider --region=ap-southeast-1 --cluster=crm-production --approve
    ```

  - Create IAM Service Account:
    ```bash
    eksctl create iamserviceaccount \
      --cluster=crm-production \
      --namespace=kube-system \
      --name=aws-load-balancer-controller \
      --role-name AmazonEKSLoadBalancerControllerRole \
      --attach-policy-arn=arn:aws:iam::392564808002:policy/AWSLoadBalancerControllerIAMPolicy \
      --approve
    ```

  - Install AWS Load Balancer Controller using Helm:
    ```bash
    helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
      --set clusterName=crm-production \
      --set serviceAccount.create=false \
      --set region=ap-southeast-1 \
      --set vpcId=vpc-02a3fe25d2c9a4844 \
      --set serviceAccount.name=aws-load-balancer-controller \
      -n kube-system
    ```

## Horizontal Pod Autoscaler (HPA)
- **Check Existing CRD HPA**:
  ```bash
  kubectl get crd
  ```

## Storage
- **EBS**:
  - Create IAM Service Account:
    ```bash
    eksctl create iamserviceaccount \
      --name ebs-csi-controller-sa \
      --namespace kube-system \
      --cluster crm-production \
      --role-name AmazonEKS_EBS_CSI_DriverRole \
      --role-only \
      --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
      --approve
    ```

## Monitoring
- **Enable Metrics Server**:
  - [Guide Link](#)

- **Prometheus**:
  - [Link](#)

- **Grafana**:
  - Add Helm Repo and Install Grafana:
    ```bash
    helm repo add grafana https://grafana.github.io/helm-charts
    helm repo update
    helm install grafana grafana/grafana --namespace monitoring --create-namespace
    ```

  - Get Admin Password:
    ```bash
    kubectl get secret --namespace monitoring grafana -o jsonpath="{.data.admin-password}" | base64 --decode ; echo
    ```

  - Set Up Port Forwarding:
    ```bash
    kubectl port-forward --namespace monitoring svc/grafana 3000:80
    ```

  - Add Prometheus Data Source:
    - Once logged in to Grafana, go to Configuration (the gear icon) and select Data Sources.
    - Click Add data source.
    - Select Prometheus.
    - In the HTTP section, set the URL to your Prometheus server. If Prometheus is deployed within the same Kubernetes cluster, it might look like `http://prometheus-server.monitoring.svc.cluster.local:9090`.
    - Click Save & Test to verify the connection.

  - Persistent Storage:
    - Create `grafana-values.yaml`:
      ```yaml
      persistence:
        enabled: true
        size: 5Gi  # Adjust the size as needed
        storageClassName: gp2  # Ensure this matches your AWS EKS storage class
        accessModes:
          - ReadWriteOnce
      ```

    - Upgrade Grafana with Persistent Storage:
      ```bash
      helm upgrade grafana grafana/grafana -f grafana-values.yaml --namespace monitoring
      ```

## Logging
- **Audit Logging**:
  - [Guide Link](#)

- **CloudWatch Logging & Monitoring**:
  - [Guide Link](#)

- **Logstash using Helm**:
  - Installation and Configuration:
    - Manage Fluentd config to parse logs to Elasticsearch.

## HashiCorp Vault
- **High Level Architecture**:
  - [URL](#)

- **Installation**:
  ```bash
  kubectl create ns vault
  kubectl config set-context --current --namespace=vault
  helm init
  helm repo add hashicorp https://helm.releases.hashicorp.com
  helm repo update
  helm install vault hashicorp/vault --set "server.dev.enabled=true"
  kubectl port-forward svc/vault 8200:8200
  ```

- **Usage**:
  ```bash
  kubectl logs $(kubectl get pods -l "app.kubernetes.io/name=vault" -o jsonpath="{.items[0].metadata.name}")
  vault login <root-token>
  vault kv put secret/myapp username=admin password=password123
  vault kv get secret/myapp
  ```
