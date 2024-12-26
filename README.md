# Implementing-Auto-scaling-with-AWS-Metrics

# Kubernetes Deployment with Horizontal Pod Autoscaling on EKS  

## Objective  
Understand the dynamics of scaling applications in Kubernetes on AWS.  

## Prerequisites  
- AWS Account  
- AWS CLI installed and configured  
- Kubectl installed  
- Eksctl installed  

## Tasks Overview  
1. Deploy an application on Amazon EKS.  
2. Implement Horizontal Pod Autoscaler (HPA).  
3. Test and verify scaling.  

## Creating an EKS Cluster  

Run the following command to create an EKS cluster:  

```bash  
eksctl create cluster --name lab3 --version 1.31 --region us-east-1 \
  --nodegroup-name App-nodes --node-type t2.micro --nodes 2 --nodes-min 1 --nodes-max 2 --managed  
```  

Update your kubeconfig for the new cluster:  

```bash  
aws eks update-kubeconfig --region us-east-1 --name lab3  
```  

## Deploying a Simple Application  

Create a YAML file named `simple-app-deployment.yaml` with the following content:  

```yaml  
apiVersion: apps/v1  
kind: Deployment  
metadata:  
  name: simple-app  
spec:  
  replicas: 1  
  selector:  
    matchLabels:  
      name: deployment  
  template:  
    metadata:  
      labels:  
        name: deployment  
    spec:  
      containers:  
      - name: app-nginx  
        image: nginx  
        ports:  
        - containerPort: 80  
        resources:  
          limits:  
            cpu: 500m  
          requests:  
            cpu: 200m  
```  

Apply the deployment:  

```bash  
kubectl apply -f simple-app-deployment.yaml  
```  

## Implement Horizontal Pod Autoscaler (HPA)  

### Step 1: Install Metrics Server  

Install the Kubernetes Metrics Server using the following command:  

```bash  
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml  
```  

### Step 2: Create the HPA  

Create a YAML file named `hpa.yaml` with the following content:  

```yaml  
apiVersion: autoscaling/v2  
kind: HorizontalPodAutoscaler  
metadata:  
  name: simple-app-hpa  
spec:  
  scaleTargetRef:  
    apiVersion: apps/v1  
    kind: Deployment  
    name: simple-app  
  minReplicas: 1  
  maxReplicas: 5  
  metrics:  
  - type: Resource  
    resource:  
      name: cpu  
      target:  
        type: Utilization  
        averageUtilization: 20  
```  

Apply the HPA configuration:  

```bash  
kubectl apply -f hpa.yaml  
```  

## Testing and Verifying Scaling  

### Step 1: Generate CPU Load  

To test the scaling, generate CPU load by running the following command inside a pod shell or from your local machine:  

```bash  
while true; do wget -q -O- http://simple-app.default.svc.cluster.local; done  
```  

### Step 2: Monitor HPA Status  

Monitor the status of the HPA to verify scaling behavior:  

```bash  
kubectl get hpa simple-app-hpa -w  
```  

### Optional: Adjusting Autoscaling Parameters  

You can adjust the autoscaling parameters as needed:  

```bash  
kubectl autoscale deployment simple-app --cpu-percent=20 --min=1 --max=10  
```  

## Cleanup  

To delete the deployment, HPA, and EKS cluster, you can use:  

```bash  
kubectl delete hpa simple-app-hpa  
kubectl delete deployment simple-app  
eksctl delete cluster --name lab3 --region us-east-1  
```  

## Conclusion  

This guide provided a step-by-step process for deploying a simple application in EKS with HPA implemented. By following these steps, you should be able to understand how scaling works in Kubernetes on AWS.