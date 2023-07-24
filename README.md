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
### Deployment Section
apiVersion: apps/v1
The apiVersion specifies the version of the Kubernetes API that this manifest uses. In this case, it uses the apps/v1 API version, which is used for working with Deployments.

kind: Deployment
The kind specifies the type of Kubernetes resource being defined in this manifest. In this case, it's a Deployment, which allows you to manage and scale a set of identical pods running a specified containerized application.

metadata

The metadata section provides metadata about the Deployment resource, such as its name and labels.

* name: mongo-express
  This sets the name of the Deployment to "mongo-express".
* labels:
  This sets labels for the Deployment, and in this case, it has the label "app: mongo-express". Labels are used for selecting and organizing resources.

spec

The spec section defines the desired state of the Deployment.

* replicas: 1
  The replicas field specifies the desired number of replicas (instances) of the Mongo Express pod to be running at any given time. In this case, it's set to 1, 
  which means only one Mongo Express pod will be deployed.
* selector:
  The selector field is used to specify the labels to identify which pods this Deployment manages. In this case, it selects pods with the label "app: mongo- 
  express".
* template:
  The template section defines the pod template that will be used for creating new pods managed by the Deployment.
  * metadata:
    The metadata field provides metadata about the pod template, including labels.
    * labels:
      This sets the label "app: mongo-express" for the pods created from this template.
 * spec:
   The spec field specifies the specification of the pods created from the template.
   * containers:
     The containers section defines the containers to run in the pod.
     * name: mongo-express
       The name of the container is set to "mongo-express".
       * image: mongo-express
         The Docker image used for the container is "mongo-express", which provides the Mongo Express web-based admin interface for MongoDB.
       * ports:
         The ports section specifies the ports that the container will listen on.
         * containerPort: 8081
           The container listens on port 8081, which is the default port for Mongo Express.
       * env:
         The env section defines environment variables to be set in the container.
         * name: ME_CONFIG_MONGODB_ADMINUSERNAME
           The environment variable ME_CONFIG_MONGODB_ADMINUSERNAME is set from a Kubernetes secret named "mongodb-secret" with the key "username". This is 
           likely used to set the Mongo Express admin username for MongoDB.
         * name: ME_CONFIG_MONGODB_ADMINPASSWORD
           The environment variable ME_CONFIG_MONGODB_ADMINPASSWORD is set from a Kubernetes secret named "mongodb-secret" with the key "password". This is 
           likely used to set the Mongo Express admin password for MongoDB.
         * name: ME_CONFIG_MONGODB_SERVER
           The environment variable ME_CONFIG_MONGODB_SERVER is set from a Kubernetes configMap named "mongodb-configmap" with the key "db-url". This is likely 
           used to specify the MongoDB server URL.
   
### Service Section
The second section in the mongo-express.yaml file is for creating a Kubernetes Service that exposes the Mongo Express Deployment externally.

kind: Service

The kind specifies the type of Kubernetes resource being defined in this manifest. In this case, it's a Service, which enables networking and load balancing for the pods.

metadata

The metadata section provides metadata about the Service resource, such as its name.            
          
* name: mongo-express-service
  This sets the name of the Service to "mongo-express-service".

spec

The spec section defines the desired state of the Service.

* selector:
  The selector field is used to specify which pods the Service should route traffic to. In this case, it selects pods with the label "app: mongo-express". 
* type: NodePort
  The type field specifies the type of Service to create. Here, it's set to "NodePort", which means the Service will be exposed on a port on each node of the 
  Kubernetes cluster.
* ports:
  The ports section specifies the ports that the Service will listen on.
  * port: 8081
    The port number that the Service listens on is set to 8081.
  * targetPort: 8081
    The targetPort is set to 8081, which means the Service will forward incoming traffic to port 8081 of the pods selected by the Service.
  * nodePort: 30000
    The nodePort specifies the port number on which the Service will be exposed on each node. In this case, it's set to 30000.

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

### Secret Section
apiVersion: v1

The apiVersion specifies the version of the Kubernetes API that this manifest uses. In this case, it uses the v1 API version, which is the core Kubernetes API version.

kind: Secret

The kind specifies the type of Kubernetes resource being defined in this manifest. In this case, it's a Secret, which is used to store sensitive data.

metadata

The metadata section provides metadata about the Secret resource, such as its name.

* name: mongodb-secret
  This sets the name of the Secret to "mongodb-secret".

type: Opaque

The type field specifies the type of Secret being created. In this case, it's set to "Opaque", which means the data stored in the Secret is treated as an arbitrary byte array and not interpreted by Kubernetes.

data

The data section is where the sensitive data is encoded and stored. The data is encoded using base64 encoding to protect it from being easily read. It's important to note that base64 encoding is an encoding method, not encryption, so it's not a secure way to protect sensitive data in the long term.

* username: c25pcg==
  This is a key-value pair in the Secret, where the key is "username" and the value is "c25pcg==". The value is base64 encoded, and when decoded, it becomes the 
  original value, which is likely the username for accessing a MongoDB database.
* password: MTIzNA==
  Similarly, this is another key-value pair in the Secret, where the key is "password" and the value is "MTIzNA==". The value is also base64 encoded and, when 
  decoded, becomes the original value, which is likely the password for accessing the MongoDB database.

* mongo-configMap.yaml
  ```
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: mongodb-configmap
  data:
    db-url: mongodb-service
### ConfigMap Section
apiVersion: v1

The apiVersion specifies the version of the Kubernetes API that this manifest uses. In this case, it uses the v1 API version, which is the core Kubernetes API version.

kind: ConfigMap

The kind specifies the type of Kubernetes resource being defined in this manifest. In this case, it's a ConfigMap, which is used to store configuration data.

metadata

The metadata section provides metadata about the ConfigMap resource, such as its name.

* name: mongodb-configmap
  This sets the name of the ConfigMap to "mongodb-configmap".

data

The data section is where the configuration data is stored as key-value pairs.

* db-url: mongodb-service
  This is a key-value pair in the ConfigMap, where the key is "db-url" and the value is "mongodb-service". This configuration data likely represents the URL or 
  connection string to access the MongoDB service.
