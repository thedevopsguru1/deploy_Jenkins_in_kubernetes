New way using helm chart:
```
helm repo add jenkinsci https://charts.jenkins.io/
```
```
helm repo update
```
```
kubectl create ns jenkins
```
```
helm install my-jenkins jenkinsci/jenkins --version 4.3.5 -n jenkins
```

helm link: https://artifacthub.io/packages/helm/jenkinsci/jenkins

#####################################################################################################



#######################################################################################################

https://www.youtube.com/watch?v=bcjOOp7x7i0&t=110s
## This below is not for the master jenkins,may be fr the workers
## To have the image  you can create a jenkins docker image from the included dockerfile.
```
docker build -t devopstrainingschool/jenkins-kubernetes .
```
### Change all the container images to the above
## First lets create namespace with name jenkins 
```
kubectl create ns jenkins
```

Now lets set namespace to jenkins by below command 
```
kubectl config view | grep namespace
kubectl config set-context --current --namespace=jenkins
```

Then apply deploymet and service which are present in master folder, if you want persistent jenkins use files present in master-Persistent
```
kubectl apply -f .
```

check jenkins-master pod is up and running or not 
```
NAME                                            READY   STATUS    RESTARTS   AGE   IP            NODE          NOMINATED NODE   READINESS GATES
pod/jenkins-master-deployment-b8cf57cdc-4cxhk   1/1     Running   0          12m   10.244.1.20   kube-node-1   <none>           <none>

NAME                             TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)                        AGE   SELECTOR
service/jenkins-master-service   NodePort   10.96.78.113   <none>        80:32147/TCP,50000:30930/TCP   12m   app=jenkins-master,group=jenkins,version=latest

NAME                                        READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS       IMAGES                              SELECTOR
deployment.apps/jenkins-master-deployment   1/1     1            1           12m   jenkins-master   deekshithsn/jenkinskubernetes:4.0   app=jenkins-master,group=jenkins,version=latest

NAME                                                  DESIRED   CURRENT   READY   AGE   CONTAINERS       IMAGES                              SELECTOR
replicaset.apps/jenkins-master-deployment-b8cf57cdc   1         1         1       12m   jenkins-master   deekshithsn/jenkinskubernetes:4.0   app=jenkins-master,group=jenkins,pod-template-hash=b8cf57cdc,version=latest
```

as you can see jenkins-master pod is running on kube-node-1 so to access jenkins get node ip and service port in my case it is 32147
```
http://kube-node-1_ip:service_port
```

Once after jenkins up and running, navigate to Manage Jenkins --> Configure System ( make sure usage set to "only build jobs with label expresions matching this node" ) --> cloud 

- In Configure cloud page select kubernetes 

![1](https://user-images.githubusercontent.com/29688323/107121878-ba13d600-68ba-11eb-8201-93e39e15d4bb.JPG)

- Configure Kubernetes details and Pod templates by clicking on each one of them 

![2](https://user-images.githubusercontent.com/29688323/107121865-b5e7b880-68ba-11eb-9423-a56ea189ad0d.JPG)

- get kubernetes url by running ``` kubectl cluster-info ``` get url and click on test connection. after clicking on it, you should see Connected Kubernetes 

![3](https://user-images.githubusercontent.com/29688323/107121870-b718e580-68ba-11eb-90ae-de903ab04eda.JPG)
![image](https://user-images.githubusercontent.com/126810742/224457953-79621efe-e90a-4190-912a-b2849f539cd1.png)

## Fix This if you have connection issue running jenkins in a k8s: or this jenkins "No valid crumb was included in the request"
```
 kubectl create clusterrolebinding permissive-binding --clusterrole=cluster-admin --user=admin --user=kubelet --group=system:serviceaccounts
```
## Fix this if you encounter issue with slave container
![image](https://user-images.githubusercontent.com/126810742/224455053-877dbc79-ea8f-466d-8587-ce554070caa5.png)

if you face any problem in this step, specially when creating the slave container:  checkout 
https://issues.jenkins.io/browse/JENKINS-55788 or https://stackoverflow.com/questions/47973570/kubernetes-log-user-systemserviceaccountdefaultdefault-cannot-get-services

- Next get service url and provide the same in jenkins. Save it 

![4](https://user-images.githubusercontent.com/29688323/107121871-b7b17c00-68ba-11eb-86c4-105e5a91d6f1.JPG)

- Next configure pod template and container details as show in below picture 
### Please update the slave docker image to : jenkins/inbound-agent

![5](https://user-images.githubusercontent.com/29688323/107121872-b84a1280-68ba-11eb-8396-37ced0cf37d1.JPG)
![6](https://user-images.githubusercontent.com/29688323/107121873-b8e2a900-68ba-11eb-949d-ddcd52bcfbf4.JPG)

- Once this is done, now create new free style job and in build step give whatever th command you want to run 

![7](https://user-images.githubusercontent.com/29688323/107121875-b8e2a900-68ba-11eb-8124-c14811a12bdf.JPG)

- Once job is created, click on build now. then you will be able to see in manage nodes a slave will be created and job will be running on that 

![8](https://user-images.githubusercontent.com/29688323/107121876-b97b3f80-68ba-11eb-9d45-0102eadc1295.JPG)
![9](https://user-images.githubusercontent.com/29688323/107121877-ba13d600-68ba-11eb-97b5-956e6c51cfd5.JPG)


