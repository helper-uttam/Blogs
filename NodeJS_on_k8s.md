# Let's Deploy our first NodeJS application on Kubernetes
Managing one or two containers can be easy for developers like us but when we are given the task to manage hundreds of containers then we might need a container orchestration tool and this gave birth to Kubernetes. And sometimes due to lack of recourses, it becomes difficult to run our data or application on Kubernetes. Here in this blog, we are going to dockerize and run a NodeJS application on Kubernetes.

## Requirement's or Prerequisite's
- NodeJS and ExpressJS fundamentals
- Familarity with Docker
- Basic knowledge of Kubernetes
  > Like- Pods, Services, Ingress, etc

## Steps to follow:
- Dockerize a NodeJS application by making a `Dockerfile` and push it to docker hub.
  > Useful Resources : [Dockerizing a Node.js web app](https://nodejs.org/en/docs/guides/nodejs-docker-webapp/)
  
  > An example of a Dockerfile can be this :
   ```yaml
    # Here, I'm using node version 14.17.0 but feel free to replace it with yours. 
    # You can check your node version with node -v
    FROM node:14.17.0
    WORKDIR /usr/src/app
    
    # Copying package.json to root directory of the virtual docker container.
    COPY package*.json ./
    
    # After copying the package.json we'll run the npm install command to install all the dependencies inside the docker virtual container
    RUN npm install
    
    # Copying everything from my directory where I've kept my Dockerfile to the docker container
    COPY . .
    
    # Here I'm exposing the port for testing purposes
    EXPOSE 3000
    
    # The name of my NodeJS application is server.js, you may have index.js or app.js, replace it as per your requirement
    CMD ["node", "server.js"]
   ```

- Make a `deployment.yaml` file that will allow describing an application's life cycle. 
  > Such as, which images to use for the app, the number of pods there should be, and the way in which they should be updated.
- Make a `serive.yaml` file that will help us in keeping track of all **running pod's IP address** so that all running pods inside that **cluster** can communicate with each other.
- Make a `ingress.yaml` file if you want to manage external user's access to the services in your kubernetes cluster.
<img src="https://github.com/helper-uttam/Blogs/blob/master/assets/service_architecture.png">

#### Enough talking! let's dive into an example 
   Let's assume we have an image of a NodeJS application that is kept on our [DockerHub](https://hub.docker.com/) registry. And in the Dockerfile we have exposed the port of the application to 3000,
   
   then all these *.yaml* files listed below might be relevant for us
   
-   `deployment.yaml` file
   
   ```yaml
    apiVersion: apps/v1
    # Specifying the kind as Deployment 
    # because we want to deploy our application
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
          # Specify your DockerHub image 
            image: <Dockerhub username/image_name>
            resources:
            # Specifying the resourses that we might need for our application
              limits:
                memory: "128Mi"
                cpu: "500m"
            ports:
            # Exposed PORT
            - containerPort: 3000
  ```
  
-   `service.yaml` file
   
   ```yaml
    apiVersion: v1
    # Specifying the kind as Service 
    # because we want to define our pod type to communicate with each other inside the cluster
    kind: Service
    metadata:
      name: test-app-svc
    spec:
      selector:
        app: test-app-1
      ports:
      - port: 3000
        targetPort: 3000
      # To create stable IP address that is accessible from nodes in the cluster
      # If we'll not define the type of service then it'll be ClusterIP by default
      type: ClusterIP  
  ```
  
 -  `ingress.yaml` file
 
  ```yaml
    apiVersion: networking.k8s.io/v1
     # Specifying the kind as Ingress 
    # because we want's to manage the external user to our service 
kind: Ingress
metadata:
# Naming our Ingress
  name: test-app-ingress
  labels:
    name: test-app-ingress
spec:
  rules:
  # Defining rules such as 
  # Providing the cluster name, DNS_NAME (where we want this image to be visible)
  - host: test-app.<DNS_NAME>
    http:
      paths:
      - pathType: Prefix
        path: "/"
        backend:
          service:
          # Specifying the name of the service that the pods are using
            name: test-app-svc
            port: 
            # Specifying the PORT to map with the service 
              number: 3000
```
 
## Deploying our application to a cluster
Make sure of the connection to the cluster where you want to deploy the application, this check can be done using 
```
kubectl service-info
```
If you are not getting your running cluster-info then you might need to configure it properly, a follow-up resource  to get this done can be [this](https://kubernetes.io/docs/tasks/access-application-cluster/access-cluster/).
If you are ready to go and your cluster is running properly then you're almost at the end of deployment. 

Now, we only have to run the `kubectl apply` command with a flag `-f` which means we are applying all of the mentioned rules in all the 3 respective files. Then it'll create pods that can be deployed to the production.
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
<table>
  <tr>
    <td> <img src="https://user-images.githubusercontent.com/72701081/146311539-96091281-5e8c-4126-8f18-9b3b9e91c3bc.gif" alt="coffee_gif"></img> </td>
    <td> <h3>Now, take a pause and drink some coffee because we've already come a long way to here.</h3> </td>
  </tr>
 </table>

After the pause make sure to check the status of each pod that we have defined in our `deployment.yaml`, In our case we have defined 3 replicas so there must be at least 3 pods visible to us.  
```
kubectl get pods
```
This command will list all the pods that are present in our cluster, check the **status of the pod**, if it shows "RUNNING" then we had successfully deployed our application to the cluster. 

If you are using **REMOTE CLUSTER** then you may cross-check your application at the host defined in `ingress.yaml`.

Or if you want to test it in your localhost then try **forwarding the port** of any of the running pods to your localhost. This can be achieved using the command 
 ```
    kubectl port-forward POD_NAME 3000:3000
 ``` 
 
Here we are forwarding the port of the service to the port 3000 of our local machine. Now we can easily test our application on  `http://localhost:3000`.
So time to sign-off with this thought

**And a big thank you for being here, hope this blog has solved each and every chunk of doubt or problem that you may have. If you took at least one of any ideas from this blog then my efforts are successful.** 

**Wish you all a very productive 2022 and blissful #HappyNewYear, May you fly high in life and success be with you always.**
