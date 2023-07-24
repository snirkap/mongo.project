# helm
The collection of Helm templates provides an efficient way to manage and deploy the entire MongoDB application stack, including the MongoDB database and the Mongo Express admin interface, on Kubernetes clusters. With Helm's templating capabilities, users can easily customize the configuration parameters, making the deployment process flexible and adaptable to different environments and requirements. Helm simplifies the management of Kubernetes deployments, enhances reusability, and streamlines the overall application deployment process.

## tutorial
* values.yaml
  ```
  # Default values for helm.
  
  replicaCount: 1
  replicaCountApp: 1
  
  image:
    db: mongo
    app: mongo-express
  
  appService:
    type: NodePort
    port: 30000
  
  dbService:
    type: ClusterIP

The values.yaml file provides default configuration values for Helm during the deployment of the MongoDB application stack:
* replicaCount: Default number of MongoDB replicas (1).
* replicaCountApp: Default number of Mongo Express replicas (1).
* image: Default Docker images for MongoDB (mongo) and Mongo Express (mongo-express).
* appService: Default service type (NodePort) and nodePort (30000) for Mongo Express.
* dbService: Default service type (ClusterIP) for MongoDB.

These values can be easily customized to scale replicas, use custom Docker images, and configure Kubernetes Services as per deployment requirements. Helm simplifies managing Kubernetes deployments, enhancing flexibility and reusability in application deployments.





* mongo.yaml
  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: {{ .Release.Name }}-mongodb-deployment
    labels:
      app: {{ .Release.Name }}-mongodb-deployment
  spec:
    replicas: {{ .Values.replicaCount }}
    selector:
      matchLabels:
        app: {{ .Release.Name }}-mongodb-deployment
    template:
      metadata:
        labels:
          app: {{ .Release.Name }}-mongodb-deployment
      spec:
        containers:
        - name: {{ .Release.Name }}-mongodb-deployment
          image: {{ .Values.image.db }}
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
    name:  {{ .Release.Name }}-mongodb-service
  spec:
    selector:
      app: {{ .Release.Name }}-mongodb-deployment
    type:   {{ .Values.dbService.type }} 
    ports:
    - name:  {{ .Release.Name }}mongodb-port
      protocol: TCP
      port:  27017
      targetPort:  27017

The transformed mongo.yaml template uses Helm's templating features to enable dynamic configuration during deployment. It allows you to provide values for the replica count, Docker image, service type, and other parameters as part of Helm chart deployment. This provides flexibility and reusability for deploying the MongoDB application with various configurations while keeping the deployment logic concise and manageable. Helm is a powerful tool for managing Kubernetes deployments, making it easier to package, distribute, and deploy complex applications on Kubernetes clusters.

The template defines a Deployment for MongoDB, where you can specify the number of replicas and the Docker image version as Helm parameters. It also sets environment variables for the MongoDB root username and password from a Kubernetes secret named "mongodb-secret".

The template also creates a Service to expose the MongoDB Deployment. The Service type can be specified during deployment using Helm parameters, allowing you to choose between "ClusterIP", "NodePort", or "LoadBalancer" types.

Helm's templating simplifies managing Kubernetes deployments, providing flexibility and reusability for deploying the MongoDB application with different configurations.

* mongo-express.yaml
  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: {{ .Release.Name }}-mongo-express-deployment
    labels:
      app: {{ .Release.Name }}-mongo-express-deployment
  spec:
    replicas: {{ .Values.replicaCountApp }}
    selector:
      matchLabels:
        app: {{ .Release.Name }}-mongo-express-deployment
    template:
      metadata:
        labels:
          app: {{ .Release.Name }}-mongo-express-deployment
      spec:
        containers:
        - name: {{ .Release.Name }}-mongo-express-deployment
          image: {{.Values.image.app}}
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
    name: {{ .Release.Name }}mongo-express-service
  spec:
    selector:
      app: {{ .Release.Name }}-mongo-express-deployment
    type:  {{ .Values.appService.type }}
    ports:
    -  port:  8081
       targetPort:  8081
       nodePort: {{ .Values.appService.port }}

The mongo-express.yaml file is a Helm template that dynamically configures the MongoDB web-based admin interface, Mongo Express, during deployment. By leveraging Helm's templating capabilities, you can easily customize the number of replicas, Docker image version, and service type according to your needs. Helm simplifies managing Kubernetes deployments, providing flexibility and reusability for deploying the MongoDB application with various configurations.

The template defines a Deployment for Mongo Express, allowing you to specify the number of replicas and the Docker image version as Helm parameters.

It also sets environment variables for the MongoDB admin username and password from a Kubernetes secret named "mongodb-secret".

The template creates a Service to expose the Mongo Express Deployment, and you can specify the service type (ClusterIP, NodePort, or LoadBalancer) and the nodePort as Helm parameters during deployment.

* mongo-configMap.yaml
  ```
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: mongodb-configmap
  data:
    db-url: {{ .Release.Name }}-mongodb-service
the ConfigMap file is like the regulare ConfigMap except the "db-url" value that configure to the new name of the mongo db service which is adapted to Release Name.

* sercret.yaml
  ```
  apiVersion: v1
  kind: Secret
  metadata:
    name: mongodb-secret
  type: Opaque
  data:
    username: c25pcg==
    password: MTIzNA==
the secret file is stay yhe same like before.
