# Running a NodeJS application on Kubernetes
The very need to make our complex application highly available, scalable, portable, and deployable in small modules independently lead to the birth of Kubernetes. And sometime due to lack to recourses it becomes difficult to run our data or application on Kubernetes. Here in this blog we are going to dockerize and run an image on Kubernetes.

## Prerequisite's
- NodeJS and ExpressJS fundamentals
- Good hold on Docker
- Basic knowledge of Kubernetes
  > Like- Pods, Services, Ingress, etc

## Steps to follow 
- [ ] Dockerize the application by making a `Dockerfile` and push it to docker hub.
  > Useful Resources : [Dockerizing a Node.js web app](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/)
- [ ] Make a `deployment.yaml` file that will allow to describe an application's life cycle. 
  > Such as, which images to use for the app, the number of pods there should be, and the way in which they should be updated.
- [ ] Make a `serive.yaml` file that will help us in keeping track of all **running pod's IP address** so that all running pods inside that **cluster** can communicate with each other.
- [ ] Make a `ingress.yaml` file if you want to manage external user's access to the services in your kubernetes cluster.

#### Example 
   Lets assume we have an image of a nodeJS application which is kept on DockerHub. And we have exposed the port to 3000 in the Dockerfile,
   
   then all these *.yaml* files can be a relevant 
   
-   `deployment.yaml` file
   
   ```
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: test-app-1
    spec:
      selector:
        matchLabels:
          app: test-app-1
      template:
        metadata:
          labels:
            app: test-app-1
        spec:
          containers:
          - name: test-app-1
            image: <Dockerhub username/image_name>
            resources:
              limits:
                memory: "128Mi"
                cpu: "500m"
            ports:
            - containerPort: 3000
  ```
  
-   `service.yaml` file
   
   ```
    apiVersion: v1
    kind: Service
    metadata:
      name: test-app-svc
    spec:
      selector:
        app: test-app-1
      ports:
      - port: 3000
        targetPort: 3000
      type: ClusterIP  
  ```
  
 -  `ingress.yaml` file
 
  ```
    apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-app-ingress
  labels:
    name: test-app-ingress
spec:
  rules:
  - host: test-app.<DNS_NAME>
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
            name: test-app-svc
            port: 
              number: 3000
```
 
## Deploying our Application to a cluster
Make sure of the connection to the cluster where you want to deploy the application, this check can be done using 
```
kubectl service-info
```
If you are not getting your running cluster info then you might need to configure it properly, a follow-up recource to get this done can be [this](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/).
If you are ready to go and your cluster is running properly then you're almost at the end of deployment. 

Now, we only have to run `kubectl apply` command with a flag `-f` which means we are applying all of the mentioned rules in all the 3 respective file. Then it'll create pods  which can be deployed to the production.
So, run all of these 3 commands in order 
```
kubectl apply -f deployment.yaml`
```
```
kubectl apply -f service.yaml
```
```
kubectl apply -f ingress.yaml
```

Now, take a pause and drink some coffee because we've already came a long way to here. After that make sure to check the status of each pods that we have defined in our `deployment.yaml`, In our case we have defined 3 replicas so there must be at least 3 pods visible to us.  
```
kubectl get pods
```
This command will list all the pods that are present in our cluster, check the **status of the pod**, if it shows "RUNNING" then we had successfully deployed our application to the cluster. 

If you are using **REMOTE CLUSTER** then you may cross check your application at the host defined in `ingress.yaml`.

Or if you want to test it in your localhost then try **forwarding the port** of any of the running pod to your localhost. This can be achieved using the command 
 ```
    kubectl port-forward POD_NAME 3000:3000
```          
