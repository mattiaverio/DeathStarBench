# Deploy a Kubernetes cluster on upcloud
## Create a Kubernetes Cluster

**Step 1: Cluster Configuration**
Log in to your UpCloud Control Panel. Navigate to **Kubernetes** and click **Create Cluster**.

* **Location:** Choose a location (e.g., `FI-HEL1` as shown below).

<img width="2339" height="1653" alt="UpCloud – New Kubernetes cluster-1" src="https://github.com/user-attachments/assets/f2625086-15f3-45d0-9b0f-6a1c09cefd35" />

**Step 2: Network**
* **Network:** Check the **Private Network** option.
> [!IMPORTANT]
> **Crucial Step:** You MUST create/select a private network (e.g., `k8s-private-net`). This connects your worker nodes securely. Note that the private network cannot be changed once the cluster has been created

<img width="2339" height="1653" alt="UpCloud – New Kubernetes cluster-2" src="https://github.com/user-attachments/assets/4a7535ab-7a37-4c36-b469-f07f6ac42f88" />

**Step 3: Node Group & SSH Configuration**
* **Plan:** Select a plan (e.g., `Development` or `General Purpose`). For this course, the `2 core, 4 GB memory` (Development) plan with **2 nodes** is sufficient.
* **Name:** Give your node group a name (e.g., `worker-group`).

<img width="2339" height="1653" alt="UpCloud – New Kubernetes cluster-3" src="https://github.com/user-attachments/assets/689642b8-35d6-4a30-8e19-79221c0d26b8" />

* **SSH Key:** It is highly recommended to add an SSH key to access your worker nodes if debugging is needed.
    * If you don't have one, follow this guide: [How to use SSH keys authentication](https://upcloud.com/docs/guides/use-ssh-keys-authentication/)
    * Select your public key in the "Authentication" section.
<img width="2339" height="1653" alt="UpCloud – New Kubernetes cluster-4" src="https://github.com/user-attachments/assets/cd090dd7-aa10-44b0-bbb4-0b561d3012f3" />


**Step 4: Create**
* **Public Access:** Ensure "Allow access from all IP addresses" is selected for the API unless you have a static IP.
Review your summary and click **Create cluster**.
> [!NOTE]
> Creating a Kubernetes cluster takes time (approximately 10 minutes). Please be patient while the status changes to `Running`.

<img width="2339" height="1653" alt="UpCloud – New Kubernetes cluster-5" src="https://github.com/user-attachments/assets/8a18a75e-d101-4994-a815-854417124144" />

## Connect to your cluster

Once the cluster status is Running, you need to configure your local environment to control it. 

**Step 1: Install kubectl**
If you haven't already, install the Kubernetes command-line tool:
[https://kubernetes.io/docs/tasks/tools/install-kubectl-linux](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)

**Step 2: Configure kubeconfig**
1.  Go to the **Kubernetes** tab in UpCloud.
2.  Scroll down to the **Kubeconfig** section.
3.  Recommended to configure the kubeconfig file manually. Click **Download kubeconfig** to save the YAML file to your local machine (e.g., `~/Downloads/coursetestcluster_kubeconfig.yaml`).
4.  Export KUBECONFIG that you downloaded above (use full path).
```bash
export KUBECONFIG=~/Downloads/coursetestcluster_kubeconfig.yaml
```
>[!TIP] 
>Running this `export` command only configures your current terminal session. To make this change permanent across all future sessions, add the command to your shell profile (`~/.bashrc` or `~/.zshrc`). Otherwise, you will need to run the command every time you open a new terminal window for this project.
<img width="2498" height="4398" alt="upcloud_kubeconfig" src="https://github.com/user-attachments/assets/61cbb4f1-d67b-4ac3-975b-2f862ba43dae" />


**Step 3: Verify Connection**
Run the following command to check if you are connected:
```bash
kubectl cluster-info
```
**Expected Output:**
```
Kubernetes control plane is running at https://xxxxxx
CoreDNS is running at https://xxxxxx
```

# Deploy hotelReservation microservice system

Clone the repo
```bash
git clone https://github.com/EvoTestOps/DeathStarBench.git
```

## Build docker images (Optional)
### Pre-requirements:
- Docker
- Docker-compose
- luarocks (apt-get install luarocks)
- luasocket (luarocks install luasocket)
### Before you start

1. Navigate to the scripts directory:
```bash
git clone https://github.com/EvoTestOps/DeathStarBench.git
cd DeathStarBench/hotelReservation/kubernetes/scripts
```
2. Build the Docker images using the provided script:
```bash
./build-docker-images.sh
```
> [!IMPORTANT]
> If you want to use your own Docker registry, you need to:
> - Open `build-docker-images.sh` and modify the `REGISTRY` variable to your Docker username
> - Update all deployment YAML files in the `kubernetes/` directory to use your modified image names
> - Example: Change `igorrudyk1/user-service:latest` to `your-username/user-service:latest`

## Deploy services

```bash
kubectl apply -Rf DeathStarBench/hotelReservation/kubernetes/
```
Wait until the deployment is complete to view the result
```bash
kubectl get pods
```

# Locust test
## Install locust
- Option.1: Using Conda
We strongly recommend using Conda virtual environment to avoid technical problems:
```bash
conda create --name hotel python=3.11
conda activate hotel
conda install -c conda-forge locust
```
- Option.2: Using pip
```bash
sudo apt-get update 
sudo apt install python3-pip
pip3 install locust
export PATH=$PATH:$HOME/.local/bin
```
## Executing the test:
Wait for the external-IP of frontend (might take up to 10 minutes):
```bash
kubectl get svc frontend -w
```
Then run the Locust test:
```bash
locust -f uh_locust_tests/locust.py --host=http://<your-external-IP-of-frontend>:5000 --headless -u 10 -r 2 -t 10s
```
_parameter description:--host means the address of host; --headless means not start the graphical interface and output the result in terminal;- u means the number of concurrent users; - r means the number of new users per second; -t means the duration of the test_

**Expected Output:** You should see statistics about request success/failure rates.

# Monitoring
For Task 1 and Task 2 reports, you need to observe the system status using these tools.
## Trace (Jaeger)
Jaeger is used for distributed tracing to monitor and troubleshoot transactions in complex distributed systems.

**Get the URL:** Watch for the external-IP of jaeger:
```bash
kubectl get svc jaeger -w
```
**Access UI:** Visit `http://<your-external-IP-of-jaeger>:16686` in your browser.

<img width="2498" height="6498" alt="jaeger UI" src="https://github.com/user-attachments/assets/b41b9a82-cdf9-4165-a0e4-2edb88ad9978" />

**How to use:**

- **Search:** Select a Service (e.g., `frontend`) and click "Find Traces".
    
- **Analyze:** Click on a specific trace to see the **Spans** (individual operations).
    
- **Identify Issues:** Look for:
    
    - **Errors:** Spans marked in red.
        
    - **Latency:** unusually long bars in the timeline.
        
    - **Waterfall view:** Helps you understand which microservice is slowing down the request.
<img width="2498" height="1602" alt="jaeger waterfall" src="https://github.com/user-attachments/assets/d3b67a1c-0f89-4d50-8bd8-5b29508dc8cc" />


## Metric (Prometheus)
Prometheus is used for event monitoring and alerting.

**Setup**

Go to the corresponding directory
```bash
cd <path-of-repo>/hotelReservation/UH_prometheus
```

Create a configmap
```bash
kubectl create configmap prometheus-config --from-file=prometheus.yml
```
Apply related resources
```bash
kubectl apply -f node-exporter-service.yaml
kubectl apply -f node-exporter.yaml
kubectl apply -f prometheus-config.yaml
kubectl apply -f prometheus-deployment.yaml
kubectl apply -f prometheus-rbac.yaml
kubectl apply -f prometheus-service.yaml
```
**Get the URL:** Gets the external IP of the prometheus service
```bash
kubectl get svc prometheus -w
```
Visit `http://<your-external-IP-of-prometheus>:9090` in your browser.
<img width="2498" height="1602" alt="prometheus main ui" src="https://github.com/user-attachments/assets/e883535f-a02d-4805-927a-ae7f01b92ea1" />

**How to use:**

- **Graph Tab:** Enter queries to visualize data over time.
    
- **Table Tab:** View the current value of metrics.

<img width="2498" height="2272" alt="prometheus UI" src="https://github.com/user-attachments/assets/1755a72d-3ff7-40f7-aeac-fd34bfb8df04" />


Some sample query metrics
- CPU utilization 
```bash
rate(node_cpu_seconds_total{mode="system"}[1m])
```
- Memory usage
```bash
node_memory_MemTotal_bytes - node_memory_MemFree_bytes

```
- Disk usage
```bash
node_filesystem_size_bytes - node_filesystem_free_bytes
```

## Logs
To debug specific pods (e.g., if a Locust test fails):
1.  Get pods name
```bash
kubectl get pods
```
2.  View the specific pod logs
```bash
kubectl logs <pod-name>
```

# Kubernetes Common Commands Sheet

## Pod Operations
```bash
# List all pods
kubectl get pods [-n namespace]

# Get pod details
kubectl describe pod <pod-name>

# Get pod logs
kubectl logs <pod-name>
kubectl logs -f <pod-name>    # Follow log output

# Execute command in pod
kubectl exec -it <pod-name> -- /bin/bash

# Delete pod
kubectl delete pod <pod-name>
```
## Service Operations
```
# List all services
kubectl get services
kubectl get svc    # Short form

# Get service details
kubectl describe service <service-name>

# Port forwarding
kubectl port-forward svc/<service-name> <local-port>:<service-port>
```
## Deployment Operations
```
# List deployments
kubectl get deployments

# Scale deployment
kubectl scale deployment <deployment-name> --replicas=<number>

# Rollout status
kubectl rollout status deployment/<deployment-name>

# Rollback deployment
kubectl rollout undo deployment/<deployment-name>
```
## Namespace Operations
```
# List namespaces
kubectl get namespaces
kubectl get ns    # Short form

# Create namespace
kubectl create namespace <namespace-name>

# Switch namespace
kubectl config set-context --current --namespace=<namespace-name>

# Delete namespace (and everything in it)
kubectl delete namespace <namespace-name>
```
> [!NOTE]
> Replace text in `<>` with your actual values.
> Add `-n <namespace>` to any command to specify a namespace.


