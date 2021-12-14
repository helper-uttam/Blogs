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
 
   Lets assume an image of a nodeJS application which is kept on DockerHub. And we have exposed the port to 3000 in the Dockerfile then this can be a required `deployment.yaml` file
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
