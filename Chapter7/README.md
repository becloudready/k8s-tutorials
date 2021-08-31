# Chapter 7: Exercises
1. Create a Pod YAML file with two containers that use the image alpine:3.12.0. Provide a command for both containers that keep them running forever
Start by generating the YAML manifest using the run command in combination with the --dry-run option:
```
kubectl run alpine --image=alpine:3.12.0 --dryrun=
client \
--restart=Never -o yaml -- /bin/sh -c "while true; do
sleep 60; \
done;" > multi-container-alpine.yaml

vim multi-container-alpine.yaml
```
After editing the Pod, the manifest could look as follows. The container names here are container1 and container2:
```
apiVersion: v1
kind: Pod
metadata:
creationTimestamp: null
labels:
run: alpine
name: alpine
spec:
containers:
- args:
- /bin/sh
- -c
- while true; do sleep 60; done;
image: alpine:3.12.0
name: container1
resources: {}
- args:
- /bin/sh
- -c
- while true; do sleep 60; done;
image: alpine:3.12.0
name: container2
resources: {}
dnsPolicy: ClusterFirst
restartPolicy: Always
status: {}
```
2. Define a Volume of type emptyDir for the Pod. Container 1 should mount the Volume to path /etc/a, and container 2 should mount the Volume to path /etc/b.
Edit the YAML file further by adding the Volume and the mount paths for both containers.
```
apiVersion: v1
kind: Pod
metadata:
creationTimestamp: null
labels:
run: alpine
name: alpine
spec:
volumes:
- name: shared-vol
emptyDir: {}
containers:
- args:
- /bin/sh
- -c
- while true; do sleep 60; done;
image: alpine:3.12.0
name: container1
volumeMounts:
- name: shared-vol
mountPath: /etc/a
resources: {}
- args:
- /bin/sh
- -c
- while true; do sleep 60; done;
image: alpine:3.12.0
name: container2
volumeMounts:
- name: shared-vol
mountPath: /etc/b
resources: {}
dnsPolicy: ClusterFirst
restartPolicy: Always
status: {}
```
Create the Pod and check if it has been created properly. You should see the Pod in Running status with two containers ready:
```
kubectl create -f multi-container-alpine.yaml
kubectl get pods
```
3. Open an interactive shell for container 1 and create the directory data in the mount path. Navigate to the directory and create the file hello.txt with the contents “Hello World.” Exit out of the container.
Use the exec command to shell into the container named container1. Create the file /etc/a/data/hello.txt with the,relevant content:
```
kubectl exec alpine -c container1 -it -- /bin/sh
/ # cd /etc/a
/etc/a # ls -l
/etc/a # mkdir data
/etc/a # cd data/
/etc/a/data # echo "Hello World" > hello.txt
/etc/a/data # cat hello.txt
Hello World
/etc/a/data # exit
```
4. Open an interactive shell for container 2 and navigate to the directory /etc/b/data. Inspect the contents of file hello.txt. Exit out of the container.
Use the exec command to shell into the container named container2. The contents of the file /etc/b/data/hello.txt should say “Hello World”:
```
kubectl exec alpine -c container2 -it -- /bin/sh
/ # cat /etc/b/data/hello.txt
Hello World
/ # exit
```
5. Create a PersistentVolume named logs-pv that maps to the hostPath /var/logs. The access mode should be ReadWriteOnce and ReadOnlyMany. Provision a storage capacity of 5Gi. Ensure that the status of the PersistentVolume shows Available.
Start by creating a new file named logs-pv.yaml. The contents could look as follows:
```
kind: PersistentVolume
apiVersion: v1
metadata:
name: logs-pv
spec:
capacity:
storage: 5Gi
accessModes:
- ReadWriteOnce
- ReadOnlyMany
hostPath:
path: /var/logs
```
Create the PersistentVolume object and check on its status:
```
kubectl create -f logs-pv.yaml
kubectl get pv
```
6. Create a PersistentVolumeClaim named logs-pvc. The access it uses is ReadWriteOnce. Request a capacity of 2Gi. Ensure that the status of the PersistentVolume shows Bound

Create the file logs-pvc.yaml to define the PersistentVolumeClaim.The following YAML manifest shows its contents:
```
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
name: logs-pvc
spec:
accessModes:
- ReadWriteOnce
resources:
requests:
storage: 2Gi
```
Create the PersistentVolume object and check on its status:
```
kubectl create -f logs-pvc.yaml
kubectl get pvc
```
7. Mount the PersistentVolumeClaim in a Pod running the image nginx at the mount path /var/log/nginx.
Create the basic YAML manifest using the --dry-run command-line option:
```
kubectl run nginx --image=nginx --dry-run=client --
restart=Never \
-o yaml > nginx-pod.yaml
```
Now, edit the file nginx-pod.yaml and bind the PersistentVolumeClaim to it:
```
apiVersion: v1
kind: Pod
metadata:
creationTimestamp: null
labels:
run: nginx
name: nginx
spec:
volumes:
- name: logs-volume
persistentVolumeClaim:
claimName: logs-pvc
containers:
- image: nginx
name: nginx
volumeMounts:
- mountPath: "/var/log/nginx"
name: logs-volume
resources: {}
dnsPolicy: ClusterFirst
restartPolicy: Never
status: {}
```
Create the Pod using the following command and check its status:
```
kubectl create -f nginx-pod.yaml
kubectl get pods
```
8. Open an interactive shell to the container and create a new file named my-nginx.log in /var/log/nginx. Exit out of the Pod.
Use the exec command to open an interactive shell to the Pod and create a file in the mounted directory:
```
kubectl exec nginx -it -- /bin/sh
```
9. Delete the Pod and re-create it with the same YAML manifest. Open an interactive shell to the Pod, navigate to the directory /var/log/nginx, and find the file you created before.
After re-creating the Pod, the file stored on the PersistentVolume should still exist:
```
kubectl delete pod nginx
kubectl create -f nginx-pod.yaml
kubectl exec nginx -it -- /bin/sh

# cd /var/log/nginx
# ls
access.log error.log my-nginx.log
# exit
```



