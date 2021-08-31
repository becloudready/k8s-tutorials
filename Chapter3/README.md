# Chapter 3: Exercises

1. Create a YAML manifest for a Pod named complex-pod. The main application container named app should use the image nginx and expose the container port 80. Modify the YAML manifest so that the Pod defines an init container named setup that uses the image busybox. The init container runs the command wget -O- google.com.

You can start by generating the YAML manifest in dry-run mode.The resulting manifest will set up the main application container:

```
kubectl run complex-pod --image=nginx --port=80 --
restart=Never \
-o yaml --dry-run=client > complex-pod.yaml
```
Edit the manifest file by adding the init container and changing some of the default settings that have been generated. The finalized manifest could look as follows:

```
apiVersion: v1
kind: Pod
metadata:
name: complex-pod
spec:
initContainers:
- image: busybox
name: setup
command: ['sh', '-c', 'wget -O- google.com']
containers:
- image: nginx
name: app
ports:
- containerPort: 80
resources: {}
dnsPolicy: ClusterFirst
restartPolicy: Never
status: {}

```
2. Create the Pod from the YAML manifest.
Run the create command to instantiate the Pod. Verify that the Pod is running without issues:
```
kubectl create -f complex-pod.yaml
kubectl get pod complex-pod
```
3. Download the logs of the init container. You should see the output of the wget command.
Use the logs command and point it to the init container to download the log output:
```
kubectl logs complex-pod -c setup
```
4. Open an interactive shell to the main application container and run the ls command. Exit out of the container.
You can target the main application as well. Here you’ll open an interactive shell and run the ls command:
```
kubectl exec complex-pod -it -c app -- /bin/sh
# ls
# exit
```
5. Force-delete the Pod.
```
kubectl delete pod complex-pod --grace-period=0 --force
```
6. Create a YAML manifest for a Pod named data-exchange. The main application container named main-app should use the image busybox. The container runs a command that writes a new file every 30 seconds in an infinite loop in the directory /var/app/data. The filename follows the pattern {counter++}-data.txt. The variable counter is incremented every interval and starts with the value 1.
You can start by generating the YAML manifest in dry-run mode.The resulting manifest will set up the main application container:
```
kubectl run data-exchange --image=busybox --
restart=Never -o yaml \
--dry-run=client > data-exchange.yaml
```
Edit the manifest file by adding the sidecar container and changing some of the default settings that have been generated. The finalized manifest could look as follows:
```
apiVersion: v1
kind: Pod
metadata:
name: data-exchange
spec:
containers:
- image: busybox
name: main-app
command: ['sh', '-c', 'counter=1; while true; do
touch \
"/var/app/data/$counter-data.txt";
counter=$((counter+1)); \
sleep 30; done']
resources: {}
dnsPolicy: ClusterFirst
restartPolicy: Never
status: {}
```

7. Modify the YAML manifest by adding a sidecar container named sidecar. The sidecar container uses the image busybox and runs a command that counts the number of files produced by the main-app container every 60 seconds in an infinite loop. The command writes the number of files to standard output.

Simply add the sidecar container alongside the main application container with the proper command. Add on to the existing YAML manifest:
```
apiVersion: v1
kind: Pod
metadata:
name: data-exchange
spec:
containers:
- image: busybox
name: main-app
command: ['sh', '-c', 'counter=1; while true; do
touch \
"/var/app/data/$counter-data.txt";
counter=$((counter+1)); \
sleep 30; done']
resources: {}
- image: busybox
name: sidecar
command: ['sh', '-c', 'while true; do ls -dq
/var/app/data/*-data.txt \
| wc -l; sleep 30; done']
dnsPolicy: ClusterFirst
restartPolicy: Never
status: {}
```
8. Define a Volume of type emptyDir. Mount the path /var/app/data for both containers.
Modify the manifest so that a Volume is used to exchange the files between the main application container and sidecar container:
```
apiVersion: v1
kind: Pod
metadata:
name: data-exchange
spec:
containers:
- image: busybox
name: main-app
command: ['sh', '-c', 'counter=1; while true; do
touch \
"/var/app/data/$counter-data.txt";
counter=$((counter+1)); \
sleep 30; done']
volumeMounts:
- name: data-dir
mountPath: "/var/app/data"
resources: {}
- image: busybox
name: sidecar
command: ['sh', '-c', 'while true; do ls -d
/var/app/data/*-data.txt \
| wc -l; sleep 30; done']
volumeMounts:
- name: data-dir
mountPath: "/var/app/data"
volumes:
- name: data-dir
emptyDir: {}
dnsPolicy: ClusterFirst
restartPolicy: Never
status: {}
```
9. Create the Pod. Tail the logs of the sidecar container.
Create the Pod, check for its existence, and tail the logs of the sidecar container. The number of files will increment over time:
```
kubectl create -f data-exchange.yaml
kubectl get pod data-exchange
kubectl logs data-exchange -c sidecar -f
```
10. Delete the Pod.
Delete the Pod:
```
kubectl delete pod data-exchange

```



