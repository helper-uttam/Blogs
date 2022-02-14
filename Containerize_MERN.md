# How to containerize a MERN stack application?
If you're the one who's having problem in containerizing a MERN stack application, in this blog we'll containerize the application together. So without further delay let's get started with the pure basic.

# What is MERN stack?
Each letter of the MERN stands for a specific technolgy, which are **M**ongoDB, **E**xpress, **R**eactJS, **N**odeJS MERN stack is a combined pack of client side application(frontend), a backend server(built with ExpressJS & NodeJS) and a database(MongoDB). 

# What are containers or, What it means to containerize an application?
Containers are nothing but a virtual environment where the image of any application can run. It enables us to test an application on our local machines  without having any prior setup, but the only pre-requisite is that we should have Docker pre-installed in our machine. There are many such tools that we can use to containerize an application but in this article we're going to use **Docker** for containerizing our application. 

# Why should I containerize?
There are many benefits of containerizing an application, but let's discuss few sole benifits:
1) It enables very quick, easy and scalable deployments.
2) It don't require any operating system, so it is independent of any operating system.

### Let's containerize the application:
We're going to setup 3 containers for Mongo, express-server and our react-app. So, let's containerize our react-app first. 
It's quite simple to dockerize a react-app with a `Dockerfile`.
```yml
  #here we're specifiying the version of node
  #we can use the :latest tag as well.
  FROM node:14-slim

  #here we're defining the working directory
  WORKDIR /user/src/app

  #copying our dependencies to the container
  COPY ./package.json ./
  COPY ./package-lock.json ./

  #installing all the dependencies that we copied 
  RUN npm install

  #copying all the files inside the container
  COPY . .

  #exposing the port so we can access it further
  EXPOSE 3000

  #starting the application
  CMD ["npm", "start"]
```
