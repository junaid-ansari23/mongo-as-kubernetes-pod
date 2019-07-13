# mongo-as-kubernetes-pod
This example to demonstrate to run the Mongo DB inside a docker container as a Kubernetes POD. I have used the Openshift container platform to run this application. This is a non-scalable Mongo DB deployment using Persistent volume claim(PVC) for storage. If we want a scalable Mongo DB(more than one replica) , StatefulSets should be used.  I will also explain how this Mongo DB can be access from a spring boot application. 