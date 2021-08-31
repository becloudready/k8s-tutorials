# Chapter 6: Exercises
1. Create a new Pod named frontend that uses the image nginx. Assign the labels tier=frontend and app=nginx. Expose the container port 80.
The fastest way to create the Pod is by using the run command.You can see in the following command that you can assign the port and labels at the time of creating the Pod:
```
kubectl run frontend --image=nginx --restart=Never --port=80 \ -l tier=frontend,app=nginx
kubectl get pods
```
2. Create a new Pod named backend that uses the image nginx. Assign the labels tier=backend and app=nginx. Expose the container port 80.
Use the same method to create the backend Pod:
```
kubectl run backend --image=nginx --restart=Never --port=80 \ -l tier=backend,app=nginx
kubectl get pods
```
3. Create a new Service named nginx-service of type ClusterIP. Assign the port 9000 and the target port 80. The label selector should use the criteria tier=backend and deployment=app.
You can use the create service command to generate the Service. Unfortunately, you cannot assign the labels right away. Therefore, you’ll write the output of the command to a YAML file and edit the label selector definition afterward:
```
kubectl create service clusterip nginx-service --tcp=9000:8081 \ --dry-run=client -o yaml > nginx-service.yaml
```
Edit the YAML manifest to modify the label selector. The result should look similar to the following YAML manifest:
```
apiVersion: v1
kind: Service
metadata:
creationTimestamp: null
labels:
app: nginx-service
name: nginx-service
spec:
ports:
- port: 9000
protocol: TCP
targetPort: 8081
selector:
tier: backend
deployment: app
type: ClusterIP
status:
loadBalancer: {}
```
Now, create the Service from the YAML file. Listing the Service should show the correct type and the exposed port:
```
kubectl create -f nginx-service.yaml
kubectl get services
```
4. Try to access the set of Pods through the Service from within the cluster. Which Pods does the Service select?
Trying to connect to the underlying Pods of the Service won’t work. For example, a wget command times out. This behavior happens because the configuration of the Service doesn’t select any Pods for two reasons. First, the label selector doesn’t match any of the existing Pods. Second, the target port isn’t available on any of
the existing Pods:
```
kubectl run busybox --image=busybox --restart=Never -it --rm -- /bin/sh
```
5. Fix the Service assignment to properly select the backend Pod and assign the correct target port.
Edit the live object of the Service to look as follows. You can see in the following code snippet that the label selector was changed,as well as the target port:
```
apiVersion: v1
kind: Service
metadata:
creationTimestamp: null
labels:
app: nginx-service
name: nginx-service
spec:
ports:
- port: 9000
protocol: TCP
targetPort: 80
selector:
tier: backend
app: nginx
type: ClusterIP
status:
loadBalancer: {}
```
As a result of the change, you will be able to connect to the backend Pod:
```
kubectl run busybox --image=busybox --restart=Never -it --rm -- /bin/sh
/ # wget --spider --timeout=1 10.110.127.205:9000
Connecting to 10.110.127.205:9000 (10.110.127.205:9000)
remote file exists
/ # exit
pod "busybox" deleted
```
6. Expose the Service to be accessible from outside of the cluster. Make a call to the Service.
You can directly modify the nginx-service live object by feeding in the desired YAML changes. Here, you’re switching from the ClusterIP to the NodePort type. You can now connect to it from outside of the cluster using the node’s IP address and the assigned static port:
```
kubectl patch service nginx-service -p \
'{ "spec": {"type": "NodePort"} }'
kubectl get services
kubectl get nodes
kubectl describe node minikube | grep InternalIP:
wget --spider --timeout=1 192.168.64.2:32682
```
7. Assume an application stack that defines three different layers: a frontend, a backend, and a database. Each of the layers runs in a Pod. You can find the definition in the YAML file app-stack.yaml. Create the namespace and the Pods using the file app-stack.yaml.
Start by creating the namespace named app-stack. Copy the contents of the provided YAML definition to the file appstack.yaml and apply it. You should end up with three Pods:
```
kubectl create namespace app-stack
kubectl apply -f app-stack.yaml
kubectl get pods -n app-stack
```
8. Create a network policy in the file app-stack-network-policy.yaml. The network policy should allow incoming traffic from the backend to the database but disallow incoming traffic from the frontend. (Won’t work on cluster)
Create a new file with the name app-stack-network-policy.yaml.The following rules describe the desired incoming and outgoing traffic for the database Pod:
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
name: app-stack-network-policy
namespace: app-stack
spec:
podSelector:
matchLabels:
app: todo
tier: database
policyTypes:
- Ingress
- Egress
ingress:
- from:
- podSelector:
matchLabels:
app: todo
tier: backend
```
Apply the YAML file using the following command:

```
kubectl create -f app-stack-network-policy.yaml
kubectl get networkpolicy -n app-stack
```
9. Reconfigure the network policy to only allow incoming traffic to the database on TCP port 3306 and no other port. (Won’t work on cluster).
You can further restrict the ports with the following definition:
```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
name: app-stack-network-policy
namespace: app-stack
spec:
podSelector:
matchLabels:
app: todo
tier: database
policyTypes:
- Ingress
- Egress
ingress:
- from:
- podSelector:
matchLabels:
app: todo
tier: backend
ports:
- protocol: TCP
port: 3306
```
The describe command can verify that the correct port ingress rule has been applied:
```
kubectl describe networkpolicy app-stack-network-policy
-n app-stack
```




