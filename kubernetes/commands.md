WHen the cluster is reset or recreated, the dashboard user and rolebinding are lost.

In order to recereate the cluster, the following commands need to be run again:
helm repo add kubernetes-dashboard https://kubernetes.github.io/dashboard/
helm upgrade --install kubernetes-dashboard kubernetes-dashboard/kubernetes-dashboard --create-namespace --namespace kubernetes-dashboard

kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard-kong-proxy 8443:443


To create a user:
kubectl apply -f dashboard-adminuser.yml
git push -u origin ma
TO asociate the user with the role:
kubectl apply -f dashboard-rolebinding.yml

TO get the token for dashboard login:
 kubectl -n kubernetes-dashboard create token admin-user

This will bring up a clean Kubernetes cluster with the dashboard installed 
and an admin user created.

Now the files in kubernetes folder can be executed.

After this the network will have started and the services will be running.

Troubleshooting:
I am getting the following error when trying to access the dashboard:

PS C:\Users\Owner\workspace\ms-service-layer-with-kubernetes\helm> kubectl -n kubernetes-dashboard port-forward svc/kubernetes-dashboard-kong-proxy 8443:443                                                                                                                                  
Forwarding from 127.0.0.1:8443 -> 8443
Forwarding from [::1]:8443 -> 8443
Handling connection for 8443
E1016 13:36:23.875541    7148 portforward.go:424] "Unhandled Error" err=<
an error occurred forwarding 8443 -> 8443: error forwarding port 8443 to pod 2b5762bd183f85c41ebfaf708c7ddc7d0f58675fd2e0d2ae4894d9849a1b6d98, uid : exit status 1: 2025/10/16 18:36:24 socat[44555] E connect(6, AF=2 127.0.0.1:8443, 16): Connection refused
>
error: lost connection to pod

This error indicates that port forwarding could not be established.

To fix it, first check if port 8443 is already in use on the local machine.
If it is, stop the process using that port or use a different port for port forwarding.

To get the logs of the pod:
kubectl -n kubernetes-dashboard logs -l app.kubernetes.io/name=kubernetes-dashboard

This is because another process is using port 8443. To find out which process is using the port:
netstat -aon | findstr :8443


Scaling
kubectl scale --replicas=n deployment/<deployment-name> -n <namespace>
This commad is applied to each of the deployments to scale them up or down.

kubectl scale deployment accounts-deployment --replicas=1

kubectl get pods will show all the running pods:

PS C:\Users\Owner\workspace\ms-service-layer-with-kubernetes\kubernetes> kubectl get pods
NAME                                       READY   STATUS    RESTARTS   AGE
accounts-deployment-75d5c8bf69-7mqg6       1/1     Running   0          8m10s
cards-deployment-69bdc85fcd-9clzr          1/1     Running   0          8m3s
configserver-deployment-76dc5dc478-msqpb   1/1     Running   0          9m47s
eurekaserver-deployment-749677c8c5-s7hlk   1/1     Running   0          9m16s
gatewayserver-deployment-b896c8dcd-hhpfx   1/1     Running   0          6m51s
keycloak-746bdb57bd-84cq8                  1/1     Running   0          82m
loans-deployment-5fbbcc5f7d-cnwjc          1/1     Running   0          7m54s
PS C:\Users\Owner\workspace\ms-service-layer-with-kubernetes\kubernetes> 

Scaling refers to the replicaset setting of the number of pods to run for each deployment.
kubectl get replicaset

PS C:\Users\Owner\workspace\ms-service-layer-with-kubernetes\kubernetes> kubectl get replicaset
NAME                                 DESIRED   CURRENT   READY   AGE
accounts-deployment-75d5c8bf69       1         1         1       9m55s
cards-deployment-69bdc85fcd          1         1         1       9m48s
configserver-deployment-76dc5dc478   1         1         1       11m
eurekaserver-deployment-749677c8c5   1         1         1       11m
gatewayserver-deployment-b896c8dcd   1         1         1       8m36s
keycloak-746bdb57bd                  1         1         1       83m
loans-deployment-5fbbcc5f7d          1         1         1       9m39s
PS C:\Users\Owner\workspace\ms-service-layer-with-kubernetes\kubernetes> 

DOnt forget to keep the yml files up to date with the scaling changes.

kubectl describe deployment <deployment-name> -n <namespace>
This command will give details of the deployment including the events.

kubectl describe deployment gatewayserver-deployment -n ms

kubectl rollout undo deployment/<deployment-name> -n <namespace>
This command will undo the last rollout change to the deployment.

Helm: package manager for Kubernetes
Individual services can be deleted with kubectl delete -f <file-name> 
but this can be tedious if there are many files. Helm can help this. It can also 
help with the Kubernetes manifest files. is is similar to apt or yum for Linux.
Helm uses charts to define, install and upgrade applications. A chart is a collection 
of files that describe a related set of Kubernetes resources.

All services can de desribed with a single helm file outlining common operations. 
This is called the service template file: 

apiVersion: v2
kind: Service
metadata:
  name: {{ .Values.deploymentLabel }}
spec:
  selector:
    app: {{ .Values.deploymentLabel }}
  type: {{ .Values.service.type }}
  ports:
    - protocol: TCP
        port: {{ .Values.service.port }}
        targetPort: {{ .Values.service.targetPort }}

The values in {{ }} are replaced by the values in the values.yaml file:

deploymentLabel: accounts-deployment
service:
    type: ClusterIP
    port: 8080
    targetPort: 8080

Helm solves these problems:
1. Manages complex Kubernetes applications
2. Manages Kubernetes manifest files
3. Manages releases of applications
4. Manages application dependencies
5. Manages application configuration
6. Manages application upgrades and rollbacks
7. Manages application versioning
8. Manages application sharing and distribution
9. Manages application lifecycle
10. Manages application templating
11. Manages application testing
12. Manages application security
13. Manages application monitoring
14. Manages application logging
15. Manages application scaling

