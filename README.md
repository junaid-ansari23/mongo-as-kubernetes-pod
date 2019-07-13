# mongo-as-kubernetes-pod
This example is to demonstrate running the Mongo DB inside a docker container as a Kubernetes POD. I have used the Openshift container platform to run this application. This is a non-scalable Mongo DB deployment using Persistent volume claim(PVC) for storage. If we want a scalable Mongo DB(more than one replica) , StatefulSets should be used.  I will also explain how this Mongo DB can be access from a spring boot application. 

Please check the file Steps.txt for detailed step running mongo POD and accessing the Mongo from a boot application.
You can find all the YML files in this repository. Deployment-config and service is a single yml.Please reach out to me at junaid.ans@gmail.com for any help.
