# Chaos Mesh & Fault injection tesing
In this task, you will validate the system reliability by running Fault-injection testing with **Chaos Mesh**. 

## 1. Install Chaos Mesh using Helm

### Prerequisites: Install Helm
From Apt (Debian/Ubuntu)
```bash
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```
If you are using a different system or for a more detailed tutorial, check out this [link](https://helm.sh/docs/intro/install/)

To check whether Helm is installed or not, execute the following command:
```bash
helm version
```

### Install Chaos Mesh
Add the Chaos Mesh repository to the Helm repository:
```bash
helm repo add chaos-mesh https://charts.chaos-mesh.org
```
View the installable versions of Chaos Mesh
```bash
helm search repo chaos-mesh
```
Create the namespace to install Chaos Mesh
```bash
kubectl create ns chaos-mesh
```
Install Chaos Mesh (using containerd runtime settings):
```bash
helm install chaos-mesh chaos-mesh/chaos-mesh -n=chaos-mesh --set chaosDaemon.runtime=containerd --set chaosDaemon.socketPath=/run/containerd/containerd.sock --version 2.7.0
```
Verify pods are running:
```bash
kubectl get po -n chaos-mesh
```

## 2. Access Chaos Dashboard
The Dashboard is exposed as a `NodePort` service. You need to access it via your worker node's Public IP and the specific NodePort.
1. **Find the Public IP of your node:**
```bash
kubectl get nodes -o wide
```
_Copy the `EXTERNAL-IP` of one of your worker nodes._

2. **Find the Port number:** 
```bash
kubectl get svc -n chaos-mesh
```
_Your output will be something like this. Look for `chaos-dashboard`. Under `PORT(S)`, find the mapped port (e.g., `2333:30820/TCP`). In this example, the port is **30820**._
```bash
NAME                            TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)                                 AGE
chaos-daemon                    ClusterIP   None             <none>        31767/TCP,31766/TCP                     22m
chaos-dashboard                 NodePort    10.140.229.113   <none>        2333:30820/TCP,2334:32143/TCP           22m
chaos-mesh-controller-manager   ClusterIP   10.131.78.187    <none>        443/TCP,10081/TCP,10082/TCP,10080/TCP   22m
chaos-mesh-dns-server           ClusterIP   10.137.78.127    <none>        53/UDP,53/TCP,9153/TCP,9288/TCP         22m
```
3. **Access via Browser:** Go to: `http://<YOUR-NODE-EXTERNAL-IP>:<PORT>`

### Login Token Generation

To utilize the Dashboard, you need a Service Account with appropriate permissions.
<img width="2498" height="1602" alt="chaosmesh1" src="https://github.com/user-attachments/assets/76c626a9-d248-4bbc-9db4-fa532048d7d0" />
**Create RBAC Configuration:** 
On the login screen, click **"Click here to generate"**. You will see the **Token Generator**. 
<img width="2498" height="2350" alt="chaosmesh_token" src="https://github.com/user-attachments/assets/a5d8c0ef-e9d1-44d3-9565-a666de50c1bc" />
**Critical Configuration Settings:**

You must configure the parameters correctly to ensure you have permission to run experiments.

* **Cluster scoped:** **CHECK this box (Recommended).**
    * *What it means:* This grants permissions across the entire Kubernetes cluster, not just a single namespace.
    * *Why:* As an administrator of this test cluster, checking this ensures you won't face "Permission Denied" errors regardless of where your pods are running.

* **Namespace:** (If you checked "Cluster scoped", this is disabled).
    * *If you did NOT check Cluster scoped:* Select `default` (where your Hotel Reservation system lives).

* **Role:** **Select `Manager`**
    * *What it means:*
        * `Viewer`: Read-only access. You can see experiments but cannot start them.
        * `Manager`: Full access. You can create, run, and delete experiments.
    * *Why:* **Do NOT select 'Viewer'** (as seen in some default screenshots). You are required to run Fault Injection tests, so you must have `Manager` privileges.
 
**Login:** Copy the token into the Chaos Dashboard login prompt. 
<img width="2498" height="2774" alt="chaosmesh_exp_ui" src="https://github.com/user-attachments/assets/0599c640-4ebc-41ab-9264-4ab41b02cfaa" />

## 3. Run Chaos Experiments (Total 5 Required)
We provide 2 example experiments. You must create **3 additional experiments** to meet the course requirements.

### Example 1: Network Delay
This injects network latency into the system.
```bash
kubectl apply -f UH_network-delay.yaml
```
### Example 2: Pod Kill
This randomly kills pods to test auto-recovery.
```
kubectl apply -f UH_pod-kill-exp.yaml
```
###  Create Your Own Experiments
You need to create YAML files for 3 more experiments. You can use the Dashboard to generate the YAML or write them manually. Please refer to the following [official website of chaos-mesh](https://chaos-mesh.org/docs/simulate-pod-chaos-on-kubernetes/) for more examples of experimental design.
> [!NOTE]
> Use the **"New Experiment"** button in the Dashboard to configure these graphically, then view/download the YAML to submit it.

## 4. Observation & Reporting

**While the experiment is running, you must generate load (run the Locust test) and observe the monitoring tools.**

1. **Start the Chaos Experiment:** 
    ```
    kubectl apply -f <your-experiment>.yaml
    ```
        
2. **Start Locust:** Run your Locust script immediately to generate traffic.
    
3. **Observe Jaeger (Traces):**
    
    - _Network Delay:_ Look for spans that become much longer than usual.
        
    - _Pod Kill:_ Look for red error spans or incomplete traces.
        
4. **Observe Prometheus (Metrics):**
    
    - _CPU Stress:_ Watch the CPU usage graph spike.
        
    - _Failures:_ Watch for a drop in successful requests or a spike in error rates.
        

**To stop an experiment:**
```
kubectl delete -f <your-experiment>.yaml
```


