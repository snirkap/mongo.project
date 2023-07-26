# mongo.project
Kubernetes Deployment with MongoDB, MongoWeb, and Helm
This repository contains Kubernetes deployment files and Helm charts for deploying MongoDB and Mongo-express applications. The project includes configuration for two deployments, two services, a Secret, and a ConfigMap.
## tutorial
### These are the commands to run the project:
1. git clone https://github.com/snirkap/mongo.project.git
2. cd mongo.project/
3. kubectl apply -f mongo-configMap.yaml
4. kubectl apply -f secret.yaml
5. kubectl apply -f mongo.yaml
6. kubectl apply -f mongo-express.yaml 
* mongo.yaml
    ```
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: mongodb
      labels:
        app: mongodb
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: mongodb
      template:
        metadata:
          labels:
            app: mongodb
        spec:
          containers:
          - name: mongodb
            image: mongo
            ports:
            - containerPort: 27017
            env:
            - name: MONGO_INITDB_ROOT_USERNAME
              valueFrom:
                secretKeyRef:
                  name: mongodb-secret
                  key: username
            - name: MONGO_INITDB_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mongodb-secret
                  key: password
    ---
    kind: Service
    apiVersion: v1
    metadata:
      name:  mongodb-service
    spec:
      selector:
        app: mongodb
      type:   ClusterIP 
      ports:
      - name:  mongodb-port
        protocol: TCP
        port:  27017
        targetPort:  27017
The "mongo.yaml" file is a Kubernetes YAML manifest that defines the necessary resources to deploy a MongoDB instance using Kubernetes. It consists of two main sections, representing a Deployment and a Service.

this is a deployment with a default mongo image, and the default port of the image expose. in the "env" section there 2 env one for root user name and one for root password whose values is taken from mongodb-secret. for the service there is a ClusterIP service with the same ports as the deployment.

* mongo-express.yaml
    ```
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: mongo-express
      labels:
        app: mongo-express
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: mongo-express
      template:
        metadata:
          labels:
            app: mongo-express
        spec:
          containers:
          - name: mongo-express
            image: mongo-express
            ports:
            - containerPort: 8081
            env:
            - name: ME_CONFIG_MONGODB_ADMINUSERNAME
              valueFrom:
                secretKeyRef:
                  name: mongodb-secret
                  key: username
            - name: ME_CONFIG_MONGODB_ADMINPASSWORD
              valueFrom:
                secretKeyRef:
                  name: mongodb-secret
                  key: password
            - name: ME_CONFIG_MONGODB_SERVER
              valueFrom:
                configMapKeyRef:
                  name: mongodb-configmap
                  key: db-url
    ---
    kind: Service
    apiVersion: v1
    metadata:
      name: mongo-express-service
    spec:
      selector:
        app: mongo-express
      type:  NodePort
      ports:
      -  port:  8081
         targetPort:  8081
         nodePort: 30000

The mongo-express.yaml file is a Kubernetes YAML manifest that defines the necessary resources to deploy a Mongo Express web-based admin interface for MongoDB using Kubernetes. It consists of two main sections, representing a Deployment and a Service.

this is a deployment with a default image of mongo-express, and the default port of the image expose. in the "env" section where a 3 env, 2 are for the user and the password of root whose values is taken from mongodb-secret and the last one is for to connect with the mongo-db whose value is taken from mongodb-configmap. for the service there is a NodePort service and the service will listen to 8081 port and the Service will forward incoming traffic to port 8081 of the pods selected by the Service. also a node port that will be exposed on each node. In this case, it's set to 30000.
 


* secret.yaml
  ```
  apiVersion: v1
  kind: Secret
  metadata:
    name: mongodb-secret
  type: Opaque
  data:
    username: c25pcg==
    password: MTIzNA==

The secret.yaml file is a Kubernetes YAML manifest that defines a Secret resource. Secrets are used to store sensitive data like passwords, access tokens, or any confidential information, and Kubernetes manages them securely. in this case is the user name and password of the root.



* mongo-configMap.yaml
  ```
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: mongodb-configmap
  data:
    db-url: mongodb-service

The configmap.yaml file is a Kubernetes YAML manifest that defines a ConfigMap resource. ConfigMaps are used to store configuration data separately from the pod specifications, making it easy to manage and modify configuration settings without changing the pod's definition. in this case is used to give the db-url for the mongo-express.


