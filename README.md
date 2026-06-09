# CST8918 - DevOps: Infrastructure as Code  

# LAB-A04 Jenkins on Kubernetes


## 1. Lab Overview

This lab was about setting up **Jenkins on my local laptop using Docker Desktop Kubernetes**. The main reason for doing it locally was to practice Jenkins jobs without using Azure cloud credits. This local setup can later help me understand how Jenkins could be deployed to a more production-like Kubernetes environment such as **Azure Kubernetes Service (AKS)**.

The lab requirement was to submit a screenshot showing:

- Jenkins Dashboard
- At least one Jenkins job that ran successfully
- My username clearly visible in the upper-right corner

My final successful job was:

```text
a04-jenkins-success-test
```

The dashboard showed:

```text
Last Success: #2
Username: Ilyas
```

---

## 2. What Jenkins Is

**Jenkins** is an open-source automation server used mostly for **CI/CD**.

CI/CD means:

- **Continuous Integration (CI):** developers push code, and Jenkins automatically builds and tests the code.
- **Continuous Delivery / Continuous Deployment (CD):** after testing, Jenkins can automatically prepare, package, or deploy the application.

In real DevOps work, Jenkins can be used to:

- pull source code from GitHub,
- build Java, Node.js, Python, or other applications,
- run unit tests and integration tests,
- build Docker images,
- push Docker images to Docker Hub or Azure Container Registry,
- deploy applications to Kubernetes,
- run infrastructure automation,
- run scripts,
- connect with tools like GitHub, Docker, Kubernetes, Maven, Gradle, npm, Terraform, Pulumi, Azure, AWS, and GCP.

In this lab, I used Jenkins only for a simple successful test job. The main learning was how Jenkins can run as a containerized application inside Kubernetes.

---

## 3. What Kubernetes Is Doing in This Lab

Kubernetes is a container orchestration platform. It is used to run and manage containers.

In this lab, Kubernetes ran Jenkins for me.

Instead of starting Jenkins directly with a normal command, I deployed Jenkins to Kubernetes using YAML files.

Kubernetes handled:

- creating the Jenkins pod,
- running the Jenkins container,
- exposing Jenkins with a service,
- keeping the Jenkins pod running,
- organizing the Jenkins resources inside a namespace.

For this lab, I used **Docker Desktop Kubernetes**, not AKS and not Minikube.

---

## 4. Tools Used

| Tool | What I Used It For |
|---|---|
| Git | Cloned the starter repository and prepared GitHub submission |
| GitHub | Starter repo source and final repo submission |
| Docker Desktop | Provided local Docker engine and local Kubernetes cluster |
| Docker | Built the custom Jenkins image |
| Kubernetes | Ran Jenkins as a pod inside a local cluster |
| kubectl | Sent commands to the Kubernetes cluster |
| YAML | Defined Kubernetes resources like Deployment, Namespace, and Service |
| Jenkins | Created and ran the successful automation job |
| Browser | Opened Jenkins at localhost |
| Brightspace | Submitted the required screenshot and/or GitHub link |

---

## 5. Starter Repository

The starter repository was:

```text
https://github.com/rlmckenney/cst8918-w24-lab-a04-jenkins-kubernetes
```

I cloned it using:

```bash
git clone https://github.com/rlmckenney/cst8918-w24-lab-a04-jenkins-kubernetes.git
cd cst8918-w24-lab-a04-jenkins-kubernetes
```

---

## 6. Files in the Repository

The repository contained these files:

```text
Dockerfile
README.md
cleanup.sh
jenkins-deployment.yaml
jenkins-namespace.yaml
jenkins-service.yaml
```

### File Summary

| File | Purpose |
|---|---|
| `README.md` | Main lab instructions |
| `Dockerfile` | Builds a custom Jenkins controller image with plugins |
| `jenkins-namespace.yaml` | Creates a Kubernetes namespace named `jenkins` |
| `jenkins-deployment.yaml` | Deploys Jenkins as a Kubernetes Deployment |
| `jenkins-service.yaml` | Exposes Jenkins through a Kubernetes Service |
| `cleanup.sh` | Deletes Jenkins resources from Kubernetes |

---

## 7. README.md Explanation

The `README.md` explains the lab goal and steps.

Main sections from the README:

1. Background and objective
2. Use Docker Desktop as the Kubernetes host
3. Customize the Jenkins container image
4. Build the Jenkins Docker image
5. Create a Jenkins namespace
6. Create and apply the Jenkins deployment
7. Create and apply the Jenkins service
8. Get the Jenkins admin password
9. Complete first login
10. Configure Jenkins Kubernetes agents
11. Test with a Jenkins job
12. Watch containers/pods
13. Fix Kubernetes permission errors if needed
14. Production considerations
15. Submission screenshot

The README says to use Docker Desktop Kubernetes and make sure the active context is:

```text
docker-desktop
```

The README also says the final submission should be a screenshot of the Jenkins Dashboard showing at least one successful job and the username visible in the upper-right corner.

---

## 8. Dockerfile

The `Dockerfile` content was:

```dockerfile
FROM jenkins/jenkins:lts-jdk17

# Add plugins for Pipelines with Blue Ocean UI and Kubernetes
RUN jenkins-plugin-cli --plugins blueocean kubernetes
```

### Explanation

```dockerfile
FROM jenkins/jenkins:lts-jdk17
```

This means the custom image starts from the official Jenkins Long-Term Support image with Java 17.

Jenkins needs Java because Jenkins is a Java-based application.

```dockerfile
RUN jenkins-plugin-cli --plugins blueocean kubernetes
```

This installs Jenkins plugins inside the image.

| Plugin | Purpose |
|---|---|
| `blueocean` | Provides a modern Jenkins pipeline UI |
| `kubernetes` | Allows Jenkins to connect with Kubernetes and use Kubernetes pods as agents |

I built the image with:

```bash
docker build -t jenkins-controller-kubernetes:1.0 .
```

This created a local image named:

```text
jenkins-controller-kubernetes:1.0
```

---

## 9. jenkins-namespace.yaml

The `jenkins-namespace.yaml` file contained:

```yaml
---
apiVersion: v1
kind: Namespace
metadata:
 name: jenkins
```

### Explanation

This file creates a Kubernetes namespace named:

```text
jenkins
```

A namespace is a logical separation inside Kubernetes. It helps keep resources organized.

In this lab, all Jenkins resources were placed in the `jenkins` namespace.

Even though this YAML file existed, during the lab I also created the namespace directly with:

```bash
kubectl create namespace jenkins
```

Both methods are valid:

```bash
kubectl create namespace jenkins
```

or:

```bash
kubectl apply -f jenkins-namespace.yaml
```

---

## 10. jenkins-deployment.yaml

The `jenkins-deployment.yaml` file created the Jenkins Deployment.

The file content was:

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
 name: jenkins
spec:
 replicas: 1
 selector:
 matchLabels:
 app: jenkins
 template:
 metadata:
 labels:
 app: jenkins
 spec:
 containers:
 - name: jenkins
 image: jenkins-controller-kubernetes:1.0
 ports:
 - containerPort: 8080
 volumeMounts:
 - name: jenkins-home
 mountPath: /var/jenkins_home
 volumes:
 - name: jenkins-home
 emptyDir: { }
```

> Note: In the actual YAML file, indentation matters. YAML must be properly indented or Kubernetes will reject it. The repository file was applied successfully in my lab.

### Explanation of Important Parts

```yaml
apiVersion: apps/v1
kind: Deployment
```

This tells Kubernetes that the resource is a Deployment.

A Deployment manages application pods and keeps the desired number of pods running.

```yaml
metadata:
 name: jenkins
```

This names the Deployment:

```text
jenkins
```

```yaml
replicas: 1
```

This means Kubernetes should run one Jenkins pod.

```yaml
selector:
 matchLabels:
 app: jenkins
```

The selector tells the Deployment which pods belong to it.

```yaml
labels:
 app: jenkins
```

The pod gets the label:

```text
app=jenkins
```

This label is also used by the Service to find the Jenkins pod.

```yaml
image: jenkins-controller-kubernetes:1.0
```

This tells Kubernetes to run the custom Jenkins image that I built with Docker.

```yaml
containerPort: 8080
```

Jenkins runs inside the container on port 8080.

```yaml
mountPath: /var/jenkins_home
```

This is where Jenkins stores its configuration, job data, plugins, and build history inside the container.

```yaml
emptyDir: { }
```

This creates temporary storage for the pod.

Important correction for learning:

The README says the volumeMounts section creates a persistent volume, but the YAML uses `emptyDir`. An `emptyDir` volume is not durable production storage. It exists while the pod exists, but it can be lost when the pod is removed. For production Jenkins, a real PersistentVolume and PersistentVolumeClaim should be used.

---

## 11. jenkins-service.yaml

The `jenkins-service.yaml` file created a Kubernetes Service.

The file content was:

```yaml
---
apiVersion: v1
kind: Service
metadata:
 name: jenkins
 namespace: jenkins
spec:
 selector:
 app: jenkins
 ports:
 - port: 80
 targetPort: 8080
 type: LoadBalancer
```

### Explanation

```yaml
kind: Service
```

This creates a Kubernetes Service.

A Service gives stable network access to pods.

```yaml
metadata:
 name: jenkins
 namespace: jenkins
```

This creates the service named `jenkins` inside the `jenkins` namespace.

```yaml
selector:
 app: jenkins
```

This tells the Service to send traffic to pods with the label:

```text
app=jenkins
```

That matches the label from the Deployment.

```yaml
port: 80
targetPort: 8080
```

This means:

```text
Service port 80 -> Jenkins container port 8080
```

```yaml
type: LoadBalancer
```

This asks Kubernetes to expose the service through a LoadBalancer.

On Docker Desktop Kubernetes, the external IP may show as `<pending>`. In my lab, this was okay because I used port-forwarding instead of relying on an external IP.

---

## 12. cleanup.sh

The `cleanup.sh` file contained:

```sh
#!/bin/sh

kubectl delete deployment jenkins -n jenkins

kubectl delete service jenkins -n jenkins

kubectl delete namespace jenkins
```

### Explanation

This script deletes the Jenkins lab resources from Kubernetes.

It deletes:

1. Jenkins Deployment
2. Jenkins Service
3. Jenkins Namespace

I can use this when I want to clean up the local Kubernetes lab.

A safer cleanup version is:

```bash
kubectl delete deployment jenkins -n jenkins --ignore-not-found=true
kubectl delete service jenkins -n jenkins --ignore-not-found=true
kubectl delete namespace jenkins --ignore-not-found=true
```

---

## 13. Problem I Had Before Starting

Before continuing the lab, Docker Desktop Kubernetes was not running.

The Docker Desktop Kubernetes screen showed:

```text
Enable Kubernetes
Create cluster
```

Because Kubernetes was not enabled, `kubectl` could not connect properly.

I also saw this error earlier:

```text
error: current-context is not set
```

This meant there was no active Kubernetes context selected at that time.

After I switched to Docker Desktop context, I still saw a DNS error:

```text
lookup kubernetes.docker.internal ... no such host
```

This happened because WSL could not resolve the Docker Desktop Kubernetes internal hostname.

The real root problem was that Docker Desktop Kubernetes had not been started/enabled yet.

---

## 14. Fixing the Kubernetes Context Problem

To avoid confusion from multiple clusters, I decided to use only Docker Desktop Kubernetes for this lab.

I did not use:

```text
minikube
AKS
```

I used:

```text
docker-desktop
```

I checked the context using:

```bash
kubectl config current-context
```

Then I switched context using:

```bash
kubectl config use-context docker-desktop
```

After Docker Desktop Kubernetes was enabled, I checked the node:

```bash
kubectl get nodes
```

The successful result showed:

```text
NAME             STATUS   ROLES           AGE   VERSION
docker-desktop   Ready    control-plane   74d   v1.34.1
```

This confirmed that my local Kubernetes cluster was ready.

---

## 15. Cleaning Previous Jenkins Resources

Before deploying Jenkins, I deleted any older Jenkins resources to avoid conflict.

Commands used:

```bash
kubectl delete namespace jenkins --ignore-not-found=true
kubectl delete clusterrolebinding jenkins --ignore-not-found=true
```

### Explanation

```bash
kubectl delete namespace jenkins --ignore-not-found=true
```

This deletes the `jenkins` namespace if it already exists.

```bash
--ignore-not-found=true
```

This prevents an error if the namespace does not exist.

```bash
kubectl delete clusterrolebinding jenkins --ignore-not-found=true
```

This deletes an old Jenkins ClusterRoleBinding if it exists.

A ClusterRoleBinding is used when Jenkins needs permission to create Kubernetes pods as agents.

---

## 16. Building the Jenkins Image

I built the custom Jenkins image:

```bash
docker build -t jenkins-controller-kubernetes:1.0 .
```

### What This Command Means

```bash
docker build
```

Builds a Docker image.

```bash
-t jenkins-controller-kubernetes:1.0
```

Tags/names the image.

```bash
.
```

Uses the current folder as the build context.

The build completed successfully.

The image included:

- Jenkins LTS with Java 17
- Blue Ocean plugin
- Kubernetes plugin

---

## 17. Creating the Jenkins Namespace

I created the namespace:

```bash
kubectl create namespace jenkins
```

The output was:

```text
namespace/jenkins created
```

This meant the namespace was created successfully.

---

## 18. Deploying Jenkins to Kubernetes

I applied the Jenkins Deployment:

```bash
kubectl apply -f jenkins-deployment.yaml -n jenkins
```

The output was:

```text
deployment.apps/jenkins created
```

This meant Kubernetes created the Jenkins Deployment.

---

## 19. Creating the Jenkins Service

I applied the Jenkins Service:

```bash
kubectl apply -f jenkins-service.yaml -n jenkins
```

The output was:

```text
service/jenkins created
```

This created a Service named:

```text
jenkins
```

inside the `jenkins` namespace.

---

## 20. Checking the Jenkins Rollout

I checked if the Jenkins Deployment was ready:

```bash
kubectl rollout status deployment/jenkins -n jenkins --timeout=300s
```

The output showed:

```text
deployment "jenkins" successfully rolled out
```

This meant Kubernetes successfully created and started the Jenkins pod.

---

## 21. Checking Kubernetes Resources

I checked all Jenkins resources:

```bash
kubectl get all -n jenkins
```

My output showed:

```text
NAME                           READY   STATUS    RESTARTS   AGE
pod/jenkins-6f499b56bf-24k6g   1/1     Running   0          3s

NAME              TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/jenkins   LoadBalancer   10.106.186.245   <pending>     80:31848/TCP   2s

NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/jenkins   1/1     1            1           3s

NAME                                 DESIRED   CURRENT   READY   AGE
replicaset.apps/jenkins-6f499b56bf   1         1         1       3s
```

### Explanation

```text
pod/jenkins-6f499b56bf-24k6g   1/1   Running
```

The Jenkins pod was running.

```text
deployment.apps/jenkins   1/1
```

The Deployment had one available pod.

```text
service/jenkins   LoadBalancer
```

The Service was created.

```text
EXTERNAL-IP <pending>
```

This was not a problem for this lab because I accessed Jenkins using port-forwarding.

---

## 22. Jenkins Pod Name

The Jenkins pod name was:

```text
jenkins-6f499b56bf-24k6g
```

This is not a password. It is the Kubernetes pod name.

I used this pod name to get the initial Jenkins admin password.

---

## 23. Getting the Jenkins Initial Admin Password

I used this command to store the Jenkins pod name in a variable:

```bash
POD=$(kubectl get pods -n jenkins -l app=jenkins -o jsonpath='{.items[0].metadata.name}')
```

Then I printed it:

```bash
echo $POD
```

Then I got the Jenkins initial admin password:

```bash
kubectl exec -n jenkins -it $POD -- cat /var/jenkins_home/secrets/initialAdminPassword
```

### Explanation

```bash
kubectl exec
```

Runs a command inside a Kubernetes pod.

```bash
-n jenkins
```

Uses the `jenkins` namespace.

```bash
-it
```

Runs interactively in the terminal.

```bash
$POD
```

Uses the Jenkins pod name.

```bash
cat /var/jenkins_home/secrets/initialAdminPassword
```

Reads the first-time Jenkins admin password file.

Important security note:

I should not save the actual password in GitHub, README files, screenshots, or public notes.

---

## 24. Accessing Jenkins in the Browser

I used port-forwarding:

```bash
kubectl port-forward -n jenkins service/jenkins 8080:80
```

### What This Means

```text
My laptop localhost:8080
        ↓
Kubernetes service/jenkins port 80
        ↓
Jenkins pod container port 8080
```

Then I opened Jenkins in the browser:

```text
http://localhost:8080
```

The port-forward terminal must stay open while I use Jenkins. If I close the terminal, Jenkins will stop being accessible from the browser, although the pod can still be running.

---

## 25. Jenkins Setup Wizard

When Jenkins opened, I completed the setup wizard.

Steps completed:

1. Entered the initial admin password.
2. Chose **Install suggested plugins**.
3. Created a Jenkins user.
4. Confirmed the Jenkins URL.
5. Started using Jenkins.

My Jenkins user was:

```text
Ilyas
```

This username needed to be visible in the screenshot.

---

## 26. Jenkins Job Created

I created a Jenkins Freestyle job named:

```text
a04-jenkins-success-test
```

A Jenkins job is an automation task.

A Freestyle job is one of the simplest Jenkins job types. It allows us to add build steps, such as shell commands.

In a real project, a Jenkins job could:

- clone a GitHub repository,
- build an application,
- run tests,
- build Docker images,
- push images to a registry,
- deploy to Kubernetes.

In this lab, the job only ran a simple shell script to prove Jenkins was working.

---

## 27. Jenkins Build Step

Inside the job, I added an **Execute shell** build step.

The script was:

```bash
echo "CST8918 Lab A04 Jenkins on Kubernetes"
echo "Student: Ilyas Zazai"
echo "Jenkins is running locally on Docker Desktop Kubernetes"
echo "This Jenkins job ran successfully"
date
echo "SUCCESS"
```

### Explanation

Each `echo` command prints text to the Jenkins console output.

The `date` command prints the current date and time.

The final `echo "SUCCESS"` helped show that the script reached the end successfully.

---

## 28. Running the Jenkins Job

I clicked:

```text
Build Now
```

Jenkins ran the job.

The console output showed:

```text
Finished: SUCCESS
```

This meant the Jenkins job successfully completed.

The Jenkins Dashboard showed:

```text
Last Success: #2
```

This meant the latest successful build was build number 2.

---

## 29. Final Screenshot

The final screenshot showed:

- Jenkins Dashboard
- The job `a04-jenkins-success-test`
- A green success icon
- `Last Success #2`
- Username `Ilyas` visible from the upper-right profile dropdown

This satisfied the submission requirement.

---

## 30. Optional README Step: Configure Jenkins Kubernetes Agents

The README also explains how to configure Jenkins to use Kubernetes agents.

A Jenkins controller manages Jenkins, but agents run jobs.

In a more advanced Kubernetes Jenkins setup:

- Jenkins controller runs as the main server.
- Jenkins creates temporary Kubernetes agent pods.
- Jobs run inside those agent pods.
- After the job finishes, the agent pod can be destroyed.

This is useful because jobs do not overload the Jenkins controller.

### Required Information for Kubernetes Cloud Configuration

The README says Jenkins needs:

1. Kubernetes controller URL
2. Internal cluster URL of the Jenkins pod

To get the Kubernetes controller URL:

```bash
kubectl cluster-info
```

Example output:

```text
Kubernetes control plane is running at https://kubernetes.docker.internal:6443
```

To get Jenkins pod details:

```bash
kubectl describe pod -n jenkins <pod_name>
```

---

## 31. Optional README Step: Configure Clouds in Jenkins UI

The README says to configure Kubernetes Cloud in Jenkins:

```text
Manage Jenkins > System Configuration > Clouds > New Cloud
```

Then:

```text
Name: local-kubernetes
Type: Kubernetes
```

Then in Kubernetes cloud details:

```text
Kubernetes URL: https://kubernetes.docker.internal:6443
Kubernetes Namespace: jenkins
Jenkins URL: http://<jenkins-pod-ip>:8080
```

Example Jenkins URL from README:

```text
http://10.1.0.54:8080
```

This tells Jenkins how to communicate with the Kubernetes cluster.

---

## 32. Optional README Step: Pod Template

The README says to create a pod template:

```text
Name: jenkins-agent
Namespace: jenkins
Labels: jenkins-agent
Usage: use this node as much as possible
```

### What a Pod Template Is

A pod template tells Jenkins what kind of Kubernetes pod to create when it needs an agent.

The label is important because Jenkins jobs can target that label.

Example:

```text
jenkins-agent
```

A Jenkins job can use that label to run on a Kubernetes agent pod.

---

## 33. Optional README Step: Update Default Executors

The README says to change the built-in node usage:

```text
Manage Jenkins > Nodes > Built-in Node > Configure
```

Change usage to:

```text
Only build jobs with label expressions matching this node
```

### Why This Matters

This prevents jobs from running directly on the Jenkins controller.

In real DevOps practice, this is important because the controller should manage Jenkins, not do heavy build work.

Jobs should run on agents.

---

## 34. Optional README Test Job for Agent Pods

The README gives this example shell script:

```bash
#!/bin/bash

for ((i=1; i<=15; i++))
do
  echo $i
  sleep 1
done
```

This script runs for 15 seconds.

The purpose is to give enough time to see the Kubernetes agent pod being created and destroyed.

---

## 35. Optional README Step: Watch Kubernetes Pods

To watch Kubernetes pods:

```bash
kubectl -n jenkins get pods --watch
```

This allows me to see pods being created in real time.

If Jenkins is configured with Kubernetes agents correctly, a new agent pod should appear when the job starts and disappear after the job completes.

---

## 36. Optional README Step: Permission Error Fix

The README mentions that Jenkins may fail to create agent pods because of a permissions error.

Example error:

```text
pods is forbidden: User "system:serviceaccount:jenkins:default" cannot create resource "pods" in API group "" in the namespace "jenkins"
```

This means the default service account in the `jenkins` namespace does not have permission to create pods.

The README fix is:

```bash
kubectl create clusterrolebinding jenkins --clusterrole cluster-admin --serviceaccount=jenkins:default
```

### Explanation

This gives the Jenkins service account high permissions in the cluster.

Important production warning:

This is okay for a local learning lab, but it is too broad for production. In production, DevOps engineers should follow least privilege and create a more restricted Role or ClusterRole.

---

## 37. What I Actually Completed vs What Is Optional

### Completed

I completed:

- Docker Desktop Kubernetes setup
- Correct Kubernetes context selection
- Jenkins custom Docker image build
- Jenkins namespace creation
- Jenkins deployment
- Jenkins service creation
- Jenkins pod verification
- Jenkins browser access using port-forward
- Jenkins first setup
- Jenkins user creation
- Jenkins Freestyle job creation
- Jenkins successful build
- Required dashboard screenshot

### Optional / For Later Learning

The README also includes Kubernetes agent configuration. I documented it above for learning, but the required screenshot was satisfied by the successful Jenkins job on the dashboard.

---

## 38. Production Considerations

This lab was a local learning setup.

For a real production Jenkins setup, I would need:

- durable persistent storage,
- secure Jenkins authentication,
- TLS/HTTPS,
- proper RBAC with least privilege,
- Jenkins agents instead of running jobs on the controller,
- monitoring and logging,
- backup and restore,
- secrets management,
- stable DNS hostname,
- persistent public/private IP depending on requirements,
- GitHub webhook support,
- scalable Kubernetes nodes,
- deployment to AKS or another managed Kubernetes platform.

The README also mentions that production would require a persistent public IP/DNS hostname for GitHub webhooks, robust persistent volume storage, and scalable host nodes.

---

## 39. Important Learning Correction: emptyDir Is Not Persistent

The README says the volumeMounts section creates persistent storage for Jenkins metadata.

However, the YAML uses:

```yaml
emptyDir: { }
```

This is temporary pod storage.

For real persistence, production Jenkins should use:

```text
PersistentVolume
PersistentVolumeClaim
StorageClass
```

On AKS, this could use Azure-managed disk or Azure Files depending on design.

So the correct understanding is:

| Storage Type | Durable? | Good For |
|---|---:|---|
| `emptyDir` | No | Temporary lab/testing |
| `PersistentVolumeClaim` | Yes | Real Jenkins data storage |
| Azure Disk / Azure Files | Yes | AKS production-style storage |

---

## 40. GitHub Push Commands

After finishing the lab, I prepared the GitHub repo.

Commands:

```bash
git status
git branch -M main
git remote remove origin
git remote add origin https://github.com/Ilyzazai/cst8918-lab-a04-jenkins-kubernetes.git
git add .
git commit -m "Complete Jenkins Kubernetes lab"
git push -u origin main
```

If `git remote remove origin` says no such remote, that is okay. It means origin was not set yet.

If Git says `nothing to commit`, that is also okay because the main lab work was running Jenkins and submitting the screenshot.

---

## 41. Cleanup Commands

To clean up Jenkins later:

```bash
kubectl delete deployment jenkins -n jenkins --ignore-not-found=true
kubectl delete service jenkins -n jenkins --ignore-not-found=true
kubectl delete namespace jenkins --ignore-not-found=true
kubectl delete clusterrolebinding jenkins --ignore-not-found=true
docker image rm jenkins-controller-kubernetes:1.0
```

If the Docker image is being used or already deleted, Docker may show an error. That is normal.

---

## 42. Full Command Summary

```bash
# Clone starter repo
git clone https://github.com/rlmckenney/cst8918-w24-lab-a04-jenkins-kubernetes.git
cd cst8918-w24-lab-a04-jenkins-kubernetes

# Use Docker Desktop Kubernetes
kubectl config use-context docker-desktop
kubectl get nodes

# Clean old resources
kubectl delete namespace jenkins --ignore-not-found=true
kubectl delete clusterrolebinding jenkins --ignore-not-found=true

# Build custom Jenkins image
docker build -t jenkins-controller-kubernetes:1.0 .

# Create namespace
kubectl create namespace jenkins

# Deploy Jenkins
kubectl apply -f jenkins-deployment.yaml -n jenkins
kubectl apply -f jenkins-service.yaml -n jenkins

# Check rollout
kubectl rollout status deployment/jenkins -n jenkins --timeout=300s

# Check resources
kubectl get all -n jenkins

# Get Jenkins pod name
POD=$(kubectl get pods -n jenkins -l app=jenkins -o jsonpath='{.items[0].metadata.name}')
echo $POD

# Get Jenkins initial admin password
kubectl exec -n jenkins -it $POD -- cat /var/jenkins_home/secrets/initialAdminPassword

# Access Jenkins locally
kubectl port-forward -n jenkins service/jenkins 8080:80
```

Browser URL:

```text
http://localhost:8080
```

---

## 43. Final Result

The lab was completed successfully.

Final result:

```text
Kubernetes context: docker-desktop
Kubernetes node: docker-desktop Ready
Jenkins namespace: jenkins
Jenkins pod: Running
Jenkins service: Created
Jenkins URL used: http://localhost:8080
Jenkins job: a04-jenkins-success-test
Build result: SUCCESS
Username visible: Ilyas
Submission evidence: Jenkins Dashboard screenshot
```

---

## 44. What I Learned From This Lab

I learned that Jenkins can be deployed as a container inside Kubernetes.

I also learned the flow of deploying a DevOps tool using Kubernetes YAML:

1. Build a Docker image.
2. Create a Kubernetes namespace.
3. Deploy the application with a Deployment.
4. Expose the application with a Service.
5. Use port-forwarding for local access.
6. Retrieve secrets/passwords from a running pod.
7. Complete application setup in the browser.
8. Create and run a Jenkins job.
9. Verify the result using Jenkins console output and dashboard.

This lab also helped me understand the difference between a local learning setup and a production setup.

For local learning, Docker Desktop Kubernetes and `emptyDir` storage are acceptable.

For production, Jenkins should use secure access, persistent storage, proper RBAC, agents, backups, monitoring, and a managed Kubernetes platform such as AKS.

