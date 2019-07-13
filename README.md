# mongo-as-kubernetes-pod
This example to demonstrate to run the Mongo DB inside a docker container as a Kubernetes POD. I have used the Openshift container platform to run this application. This is a non-scalable Mongo DB deployment using Persistent volume claim(PVC) for storage. If we want a scalable Mongo DB(more than one replica) , StatefulSets should be used.  I will also explain how this Mongo DB can be access from a spring boot application. 

Here are the steps:
1.	Create a Persistent volume claim(PVC) in openshift cluster. 
Sample YML:-
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mongo-pv-storage
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1G
  storageClassName: standard

create volume using openshift cli:-
oc create –f <pvc>.yml

2.	Create deployment config
Sample YML:
apiVersion: v1
kind: DeploymentConfig
metadata:
  name: mongodb
  labels:
    name: mongodb
spec:
  template:
    metadata:
      labels:
        name: mongodb
      annotations:
        app_version: latest
    spec:
      containers:
      - name: mongodb-pod
        image: registry.access.redhat.com/rhscl/mongodb-32-rhel7
        ports:
        - containerPort: 27017
          protocol: TCP
        env:
        - name: MONGODB_USER
          value: DEMO
        - name: MONGODB_PASSWORD
          value: DEMO@123
        - name: MONGODB_DATABASE
          value: demodatabase
        - name: MONGODB_ADMIN_PASSWORD
          value: my@dmIn
        resources:
          limits:
            cpu: 500m
            memory: 1Gi
          requests:
            cpu: 100m
            memory: 500Mi
        volumeMounts:
        - name: pv
          mountPath: /var/lib/mongodb/data
      volumes:
      - name: pv
        persistentVolumeClaim:
          claimName: mongo-pv-storage
  replicas: 1
  selector:
    name: mongodb
  triggers:
  - type: ConfigChange
  strategy:
    type: Rolling
    rollingParams:
      updatePeriodSeconds: 1
      intervalSeconds: 1
      timeoutSeconds: 1000

I am using image - registry.access.redhat.com/rhscl/mongodb-32-rhel7 for deployment. Please check for Mongo credentials –user, password, database. This is required for mongo connectivity from other application. In the PVC section , we will mention that pvc that we created in first step.

3.	Creating service
Sample YML:
apiVersion: v1
kind: Service
metadata:
  name: mongodb-service
  labels:
    name: mongodb-service
spec:
  ports:
  - port: 27017
    name: port2
    protocol: TCP
    targetPort: 27017
  selector:
    name: mongo
In this example, I have combined deployment and service into single YML file.

Creating deployment & service with single YML
Oc create –f <deployment-service>.yml

If all goes good we will have Mongo DB running inside a new POD created in kubenetes cluster. You can check the log.

4.	Connecting to Mongo DB pod from an application
Important thing to notice here is that, other application can access this Mongo DB only as localhost and must be deployed in the same Namespace and cluster. Unlike a web application that can be exposed outside a cluster using opershift routes or other object, Mongo DB cannot be accessed that way. Let’s take the example of spring boot application that will connect to Mongo.Both Mongo and boot application should be running inside same namespace.

Steps:
4.1	Deployment config of our application should have a environment variable for mongodb service create in steps 2 & 3. Sample YML:

kind: DeploymentConfig
apiVersion: v1
metadata:
  name: boot-app
spec:
  template:
    metadata:
      labels:
        name: boot-app
      annotations:
        app_version: latest
    spec:
      containers:
      - name: boot-app-pod
        image: <my_docker_image>
        imagePullPolicy: Always
        ports:
        - containerPort: 8080
          protocol: TCP
        env:
        - name: MONGODB_SERVICE_HOST
          value: mongodb-service

4.2	boot application.yml snippets for mongo connection details
spring:
  profiles: default  
  cache:
    caffeine:
      spec: maximumSize=500, expireAfterAccess=36000s  
  data:
    mongodb:
      uri: mongodb://DEMO:DEMO@123@${MONGODB_SERVICE_HOST}:27017/demodatabase

		Note for ${MONGODB_SERVICE_HOST}, this is pointing to your mongo service.

You can find all the YML files in this repository. Deployment-config and service is a single yml . Please reach out to me at junaid.ans@gmail.com for any help.
