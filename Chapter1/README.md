1. Create a new Pod named nginx running the image nginx:1.17.10. Expose the container port 80. The Pod should live in the namespace named ckad.


```
kubectl create namespace ckad
kubectl run nginx --image=nginx:1.17.10 --port=80 --namespace=ckad

```
2. Get the details of the Pod including its IP address

```
kubectl get pod nginx --namespace=ckad -o wide

```

3. . Create a temporary Pod that uses the busybox image to execute a wget command inside of the container. The wget command should access the endpoint exposed by the nginx container. You should see the HTML response body rendered in the terminal.

```
kubectl run busybox --image=busybox --restart=Never --
rm -it -n ckad \
-- wget -O- 10.1.0.66:80

```
4. Get the logs of the nginx container.

```
kubectl logs nginx --namespace=ckad

```
5. Add the environment variables DB_URL=postgresql://mydb:5432 and DB_USERNAME=admin to the container of the nginx Pod.

You will have to re-create the object with a modified YAML file,but first you’ll have to delete the existing object:

```
kubectl delete pod nginx --namespace=ckad

```
Edit the existing YAML file nginx-pod.yaml:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.17.10
    ports:
    - containerPort: 80
    env:
    - name: DB_URL
      value: postgresql://mydb:5432
    - name: DB_USERNAME
      value: admin

```
Apply the changes:

```
kubectl create -f nginx-pod.yaml --namespace=ckad

```


6. Open a shell for the nginx container and inspect the contents of the current directory ls -l.

```
kubectl exec -it nginx --namespace=ckad -- /bin/sh
# ls -l
```

7. Create a YAML manifest for a Pod named loop that runs the busybox image in a container. The container should run the following command: for i in {1..10}; do echo "Welcome $i times"; done. Create the Pod from the YAML manifest. What’s the status of the Pod?

```
kubectl run loop --image=busybox -o yaml --dryrun=
client \
--restart=Never -- /bin/sh -c 'for i in 1 2 3 4 5 6 7 8
9 10; \
do echo "Welcome $i times"; done' \
> pod.yaml

kubectl create -f pod.yaml --namespace=ckad

```

The status of the Pod will say Completed, as the executed command in the container does not run in an infinite loop:

```
kubectl get pod loop --namespace=ckad

```
8. Edit the Pod named loop. Change the command to run in an endless loop. Each iteration should echo the current date

```
kubectl delete pod loop --namespace=ckad

```
Change the YAML file content:

```
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: loop
  name: loop
spec:
  containers:
  - args:
    - /bin/sh
    - -c
    - while true; do date; sleep 10; done
    image: busybox
    name: loop
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}

```
Create the Pod from the YAML file:
```
kubectl create -f pod.yaml --namespace=ckad

```
9. Inspect the events and the status of the Pod loop.
```
kubectl describe pod loop --namespace=ckad | grep -C 10

```
10. Delete the namespace ckad and its Pods.
```
kubectl delete namespace ckad

```




