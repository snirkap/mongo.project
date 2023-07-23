# mongo.project
Kubernetes Deployment with MongoDB, MongoWeb, and Helm
This repository contains Kubernetes deployment files and Helm charts for deploying MongoDB and Mongo-express applications. The project includes configuration for two deployments, two services, a Secret, and a ConfigMap.
## tutorial
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
### Deployment Section
apiVersion: apps/v1
The apiVersion specifies the version of the Kubernetes API that this manifest uses. In this case, it uses the apps/v1 API version, which is used for working with Deployments.

kind: Deployment
The kind specifies the type of Kubernetes resource being defined in this manifest. In this case, it's a Deployment, which allows you to manage and scale a set of identical pods running a specified containerized application.

metadata
The metadata section provides metadata about the Deployment resource, such as its name and labels.
* name: mongodb
This sets the name of the Deployment to "mongodb".
* labels:
This sets labels for the Deployment, and in this case, it has the label "app: mongodb". Labels are used for selecting and organizing resources.

spec
The "spec" section defines the desired state of the Deployment.
* replicas: 1
The "replicas" field specifies the desired number of replicas (instances) of the MongoDB pod to be running at any given time. In this case, it's set to 1, which means only one MongoDB pod will be deployed.
* selector:
The "selector" field is used to specify the labels to identify which pods this Deployment manages. In this case, it selects pods with the label "app: mongodb".
* template:
The "template" section defines the pod template that will be used for creating new pods managed by the Deployment.
  * metadata:
  The "metadata" field provides metadata about the pod template, including labels.
      * labels:
      This sets the label "app: mongodb" for the pods created from this template.
  * spec:
  The spec field specifies the specification of the pods created from the template.
      * containers:
        The containers section defines the containers to run in the pod.
        * name: mongodb
          The name of the container is set to "mongodb".
          * image: mongo
            The Docker image used for the container is "mongo", which is the official MongoDB image from Docker Hub.
          * ports:
            The ports section specifies the ports that the container will listen on.
            * containerPort: 27017
              The container listens on port 27017, which is the default port for MongoDB.
          * env:
            The "env" section defines environment variables to be set in the container.
            * name: MONGO_INITDB_ROOT_USERNAME
              The environment variable MONGO_INITDB_ROOT_USERNAME is set from a Kubernetes secret named "mongodb-secret" with the key "username". This is likely                used to set the MongoDB root user's username.
            * name: MONGO_INITDB_ROOT_PASSWORD
              The environment variable MONGO_INITDB_ROOT_PASSWORD is set from a Kubernetes secret named "mongodb-secret" with the key "password". This is likely                used to set the MongoDB root user's password.
### Service Section
The second section in the mongo.yaml file is for creating a Kubernetes Service that exposes the MongoDB Deployment internally within the Kubernetes cluster.

kind: Service
The kind specifies the type of Kubernetes resource being defined in this manifest. In this case, it's a Service, which enables networking and load balancing for the pods.

metadata
The metadata section provides metadata about the Service resource, such as its name.
* name: mongodb-service
  This sets the name of the Service to "mongodb-service".

spec
The spec section defines the desired state of the Service.
* selector:
  The selector field is used to specify which pods the Service should route traffic to. In this case, it selects pods with the label "app: mongodb".
* type: ClusterIP
  The type field specifies the type of Service to create. Here, it's set to "ClusterIP", which means the Service will be accessible only within the Kubernetes      cluster and will get an internal IP address.
* ports:
  The ports section specifies the ports that the Service will listen on.
  * name: mongodb-port
    This sets the name of the port to "mongodb-port".
  * protocol: TCP
    The port's protocol is set to TCP.
  * port: 27017
    The port number that the Service listens on is set to 27017.
  * targetPort: 27017
    The targetPort is set to 27017, which means the Service will forward incoming traffic to port 27017 of the pods selected by the Service.


            
          

  
  
  
  




