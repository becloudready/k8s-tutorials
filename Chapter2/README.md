# Chapter 2: Exercises

1. Create a directory with the name config. Within the directory, create two files. The first file should be named db.txt and contain the key-value pair password=mypwd. The second file is named ext-service.txt and should define the key-value pair api_key=LmLHbYhsgWZwNifiqaRorH8T.

```
mkdir config
echo -e "password=mypwd" > config/db.txt
echo -e "api_key=LmLHbYhsgWZwNifiqaRorH8T" > config/ext-service.txt

```
2. Create a Secret named ext-service-secret that uses the directory as data source and inspect the YAML representation of the object.
```
kubectl create secret generic ext-service-secret --from-file=config
kubectl get secret ext-service-secret -o yaml
```
3. Create a Pod named consumer with the image nginx and mount the Secret as a Volume with the mount path /var/app. Open an interactive shell and inspect the values of the Secret.

```
kubectl run consumer --image=nginx --dry-run=client --restart=Never \ -o yaml > pod.yaml
```
Next, modify the manifest by mounting the Secret as a Volume.
```
apiVersion: v1
kind: Pod
metadata:
creationTimestamp: null
labels:
run: consumer
name: consumer
spec:
containers:
- image: nginx
name: consumer
volumeMounts:
- name: secret-volume
mountPath: /var/app
readOnly: true
resources: {}
volumes:
- name: secret-volume
secret:
secretName: ext-service-secret
dnsPolicy: ClusterFirst
restartPolicy: Never
status: {}
```
Now, create the Pod. Shell into the Pod as soon as the status indicates Running. Navigate to the directory /var/app.
```
kubectl create -f pod.yaml
kubectl get pod consumer
kubectl exec consumer -it -- /bin/sh
cd /var/app
ls
cat db.txt
cat ext-service.txt
```

4. Use the declarative approach to create a ConfigMap named ext-service-configmap. Feed in the key-value pairs api_endpoint=https://myapp.com/api and username=bot as literals.
```
apiVersion: v1
kind: ConfigMap
metadata:
name: ext-service-configmap
data:
api_endpoint: https://myapp.com/api
username: bot
```
With this definition, create the object:
```
kubectl create -f configmap.yaml
kubectl get configmap ext-service-configmap
kubectl get configmap ext-service-configmap -o yaml
```

5. Inject the ConfigMap values into the existing Pod as environment variables. Ensure that the keys conform to typical naming conventions of environment variables.


You will have to re-create the Pod to make the necessary changes as Kubernetes doesn’t allow adding new environment variables to a running container. The resulting YAML manifest could look like the following code snippet:
```
apiVersion: v1
kind: Pod
metadata:
creationTimestamp: null
labels:
run: consumer
name: consumer
spec:
containers:
- image: nginx
name: consumer
volumeMounts:
- name: secret-volume
mountPath: /var/app
readOnly: true
env:
- name: API_ENDPOINT
valueFrom:
configMapKeyRef:
name: ext-service-configmap
key: api_endpoint
- name: USERNAME
valueFrom:
configMapKeyRef:
name: ext-service-configmap
key: username
volumes:
- name: secret-volume
secret:
secretName: ext-service-secret
dnsPolicy: ClusterFirst
restartPolicy: Always
status: {}
```
6. Open an interactive shell and inspect the values of the ConfigMap

```
kubectl exec -it consumer -- /bin/sh

```
7. Define a security context on the container level of a new Pod named security-context-demo that uses the image alpine. The security context adds the Linux capability CAP_SYS_TIME to the container. Explain if the value of this security context can be redefined in a Pod-level security context.

```
kubectl run security-context-demo --image=alpine --dryrun=client \ --restart=Never -o yaml > pod.yaml

```
Edit the file pod.yaml and add the security context.
```
apiVersion: v1
kind: Pod
metadata:
creationTimestamp: null
labels:
run: security-context-demo
name: security-context-demo
spec:
containers:
- image: alpine
name: security-context-demo
resources: {}
securityContext:
capabilities:
add: ["SYS_TIME"]
dnsPolicy: ClusterFirst
restartPolicy: Never
status: {}
```
8. Define a ResourceQuota for the namespace project-firebird. The rules should constrain the count of Secret objects within the namespace to 1.

Start by creating the new namespace:
```
kubectl create namespace project-firebird
kubectl get namespace project-firebird
```

Create a YAML manifest for the ResourceQuota. You can define the maximum count of Secrets in the namespace with the attribute spec.hard.secrets:
```
apiVersion: v1
kind: ResourceQuota
metadata:
name: firebird-quota
spec:
hard:
secrets: 1
```
Say you stored the manifest in the file resource-quota.yaml; you can create it with the following command.

```
kubectl create -f resource-quota.yaml --namespace=project-firebird
```

9. Create as many Secret objects within the namespace until the maximum number enforced by the ResourceQuota has been reached.
```
kubectl get resourcequota firebird-quota --namespace=project-firebird

kubectl get secrets --namespace=project-firebird
```

Now, go ahead and create a Secret. The ResourceQuota will render an error message and disallow the creation of the Secret:

```
kubectl create secret generic my-secret --fromliteral=test=hello \ --namespace=project-firebird
```

10. Create a new Service Account named monitoring and assign it to a new Pod with an image of your choosing. Open an interactive shell and locate the authentication token of the assigned Service Account.
```
kubectl create serviceaccount monitoring
kubectl get serviceaccount monitoring
kubectl run nginx --image=nginx --restart=Never \
--serviceaccount=monitoring
kubectl exec -it nginx -- /bin/sh
cat /var/run/secrets/kubernetes.io/serviceaccount/token
```


