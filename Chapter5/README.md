# Chapter 5: Exercises
1. Create three Pods that use the image nginx. The names of the Pods should be pod-1, pod-2, and pod-3. Assign the label tier=frontend to pod-1 and the label tier=backend to pod-2 and pod-3. All pods should also assign the label team=artemidis.

Start by creating the Pods. You can assign labels at the time of creation:
```
kubectl run pod-1 --image=nginx --restart=Never \
--labels=tier=frontend,team=artemidis
kubectl run pod-2 --image=nginx --restart=Never \
--labels=tier=backend,team=artemidis
kubectl run pod-3 --image=nginx --restart=Never \
--labels=tier=backend,team=artemidis
kubectl get pods --show-labels
```
2. Assign the annotation with the key deployer to pod-1 and pod-3. Use your own name as the value.
You can either edit the live objects to add an annotation or use the annotate command. We’ll use the imperative command here:
```
kubectl annotate pod pod-1 pod-3 deployer='Benjamin Muschko'
kubectl describe pod pod-1 pod-3 | grep Annotations:
```
3. From the command line, use label selection to find all Pods with the team artemidis or aircontrol and that are considered a backend service.
The label selection requires you to combine equality- and set-based criteria to find the Pods:
```
kubectl get pods -l tier=backend,'team in
(artemidis,aircontrol)' \
--show-labels
```
4. Create a new Deployment named server-deployment. The Deployment should control two replicas using the image grand-server:1.4.6.
The create deployment command creates a Deployment but doesn’t allow for providing the number of replicas as a commandline option. You will have to run the scale command afterward:
```
kubectl create deployment server-deployment --image=grand-server:1.4.6
```
5. Inspect the Deployment and find out the root cause for its failure.
You will find that the Deployment doesn’t make any of its Pods available even after waiting for a while. The problem is that the assigned image does not exist. Having a look at one of its Pods will reveal the issue in the events log:
```
kubectl get deployments
kubectl get pods
kubectl describe pod server-deployment-779f77f555-q6tq2
```
6. Fix the issue by assigning the image nginx instead. Inspect the rollout history. How many revisions would you expect to see?
The set image command is a handy shortcut for assigning a new image to a Deployment. After the change, the rollout history should contain two revisions: one revision for the initial creation of the Deployment and another one for the change to the image:
```
kubectl set image deployment server-deployment grandserver=nginx
kubectl rollout history deployments server-deployment
```
7. Create a new CronJob named google-ping. When executed, the Job should run a curl command for google.com. Pick an appropriate image. The execution should occur every two minutes.
You can use the image nginx, which has the command-line tool curl installed. The Unix cron expression for this job is */2 * * * *:
```
kubectl create cronjob google-ping --schedule="*/2 * *
* *" \
--image=nginx -- /bin/sh -c 'curl google.com'
```
8. Show the execution of the cronjob and tail it. 
You can inspect when a CronJob is executed using the -w command-line option:
```
kubectl get cronjob -w
```
9. Reconfigure the CronJob to retain a history of seven executions.
Explicitly assign the value 7 to the spec.successfulJobsHistoryLimit attribute of the live object. The resulting YAML manifest should have the following configuration:
```
spec:
successfulJobsHistoryLimit: 7
```
10. Reconfigure the CronJob to disallow a new execution if the current execution is still running. Consult the Kubernetes documentation for more information.
Edit the default value of spec.concurrencyPolicy of the live object. The resulting YAML manifest should have the following configuration:
```
spec:
concurrencyPolicy: Forbid
```
