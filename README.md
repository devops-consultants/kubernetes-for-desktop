# Kubernetes on Docker-for-Desktop
Extra deployments to finish off your kubernetes installation on Docker-for-Desktop deployments

After enabling Kubernetes in the Docker-for-Desktop preferences, you will have a basic kubernetes setup running locally on your docker instance. It does not however deploy the kubernetes dashboard, or any Ingress Controller / Service.

These additional steps will 
* deploy the Traefik Ingress Controller, 
* expose the Traefic Admin Console,
* deploy the kubernetes dashboard,
* configure an Ingress service to expose the kubernetes dashboard

## Deploy Traefik Ingress
From your prompt, ensure your kubernetes cluster is running `kubectl cluster-info`

Deploy Traefik by running:
```
$ kubectl create -f https://github.com/devops-consultants/kubernetes-for-desktop/raw/master/traefik-rbac.yaml
$ kubectl create -f https://github.com/devops-consultants/kubernetes-for-desktop/raw/master/traefik-deployment.yaml
```
Once the traefik pod has finished deploying, you can confirm what ports the Traefik Ingress Service is listening on by running:
```
$ kubectl get svc -n kube-system traefik-ingress-service
NAME                      TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)                       AGE
traefik-ingress-service   NodePort   10.109.59.180   <none>        80:31581/TCP,8080:30417/TCP   15h
```
In this example, the ingress service is exposed on port 31581 and the admin console on 30417 - check you can view the current Ingress configuration by going to http://localhost:adminPort/ ie. http://localhost:30417/
  
## Deploy Kubernetes Dashboard
Deploy the dashboard by running:
```
kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/master/src/deploy/recommended/kubernetes-dashboard.yaml
```
After deploying the dashboard, we need to make a small modification to the definition of the kubernetes-dashboard service, giving the port the name 'https'. This is so that Traefik knows to use a https connection from the Ingress Controller to the dashboard deployment.
```
kubectl apply -f https://github.com/devops-consultants/kubernetes-for-desktop/raw/master/kubernetes-dashboard-service.yml
```

## Deploy Ingress Rules
This ingress rule assumes that we are using a hostname of k8s.local to access the local kubernetes installation on Docker-for-Desktop. Add the following to your `/etc/hosts` file:
```
127.0.0.1    k8s.local
```
As an example of ingress rules, the following will also add an ingress rule to the Traefik admin console. Run:
```
kubectl create -f https://github.com/devops-consultants/kubernetes-for-desktop/raw/master/dashboard-ingress.yml
```
You should now be able to access:
* http://k8s.local:ingressPort/dashboard, ie. http://k8s.local:31581/dashboard
* http://k8s.local:ingressPort/ingress-admin
