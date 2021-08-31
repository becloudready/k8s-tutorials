# Chapter 4: Exercises
1. Define a new Pod named web-server with the image nginx in a YAML manifest. Expose the container port 80. Do not create the Pod yet.
You can start by generating the YAML manifest in dry-run mode.The resulting manifest will create the container with the proper image:
```
kubectl run web-server --image=nginx --port=80 --
restart=Never \
-o yaml --dry-run=client > probed-pod.yaml
```
2. For the container, declare a startup probe of type httpGet. Verify that the root context endpoint can be called. Use the default configuration for the probe.
Edit the manifest by defining a startup probe. The finalized manifest could look as follows:
```
apiVersion: v1
kind: Pod
metadata:
creationTimestamp: null
labels:
run: web-server
name: web-server
spec:
containers:
- image: nginx
name: web-server
ports:
- containerPort: 80
name: nginx-port
startupProbe:
httpGet:
path: /
port: nginx-port
resources: {}
dnsPolicy: ClusterFirst
restartPolicy: Never
status: {}
```
3. For the container, declare a readiness probe of type httpGet. Verify that the root context endpoint can be called. Wait five seconds before checking for the first time.
Further edit the manifest by defining a readiness probe. The finalized manifest could look as follows:
```
apiVersion: v1
kind: Pod
metadata:
creationTimestamp: null
labels:
run: web-server
name: web-server
spec:
containers:
- image: nginx
name: web-server
ports:
- containerPort: 80
name: nginx-port
startupProbe:
httpGet:
path: /
port: nginx-port
readinessProbe:
httpGet:
path: /
port: nginx-port
initialDelaySeconds: 5
resources: {}
dnsPolicy: ClusterFirst
restartPolicy: Never
status: {}
```
4. For the container, declare a liveness probe of type httpGet. Verify that the root context endpoint can be called. Wait 10 seconds before checking for the first time. The probe should run the check every 30 seconds.
Further edit the manifest by defining a liveness probe. The finalized manifest could look as follows:
```
apiVersion: v1
kind: Pod
metadata:
creationTimestamp: null
labels:
run: web-server
name: web-server
spec:
containers:
- image: nginx
name: web-server
ports:
- containerPort: 80
name: nginx-port
startupProbe:
httpGet:
path: /
port: nginx-port
readinessProbe:
httpGet:
path: /
port: nginx-port
initialDelaySeconds: 5
livenessProbe:
httpGet:
path: /
port: nginx-port
initialDelaySeconds: 10
periodSeconds: 30
resources: {}
dnsPolicy: ClusterFirst
restartPolicy: Never
status: {}
```
5. Create the Pod and follow the lifecycle phases of the Pod during the process.
Create the Pod, then check its READY and STATUS columns. The container will transition from ContainerCreating to Running. At some point, 1/1 container will be available:
```
kubectl create -f probed-pod.yaml
kubectl get pod web-server
```
6. Inspect the runtime details of the probes of the Pod.
You should find the configuration of the probes when executing the describe command:
```
kubectl describe pod web-server
```
7. Retrieve the metrics of the Pod (e.g., CPU and memory) from the metrics server.
Run the top command to retrieve monitoring metrics from the metrics server:
```
kubectl top pod web-server
```
8. Create a Pod named custom-cmd with the image busybox. The container should run the command top-analyzer with the command-line flag --all
You can use the run command and provide the command to run as an argument. The status of the Pod will turn out to be Error:
```
kubectl run custom-cmd --image=busybox --restart=Never
\
-- /bin/sh -c "top-analyzer --all"

kubectl get pod custom-cmd
```
9. Inspect the status. How would you further troubleshoot the Pod to identify the root cause of the failure.
Use the logs command to find more useful runtime information.From the error message, you’ll know that the tool topanalyzer isn’t available for the image:
```
kubectl logs custom-cmd
```


