## Deploying web app on k8

### Requirement
* high availability
* fault tolerance
* easily scalable
* platform independent
* portable & flexible

### Steps
* kubernetes cluster
* containarised apps
* create EBS volume for DB pod
* LABEL nodes with zones names

1. write k8 definition file to create objects on k8 cluster
2. we use
   * deployment
   * service
   * secret
   * volume


create a git repo. ther we keep all the k8 definition file
1. downalod intellij idea, to track and interact git directly
2. install kubernetes plugin
3. click on get from version control > paste git repo link


### Practical
1. create ec2 instance and install kpops
follow: https://github.com/KarampudiKarthik/Devops/blob/main/Kubernetes/4.%20setup%20kops.md
2. create k8 cluster
```
kops create cluster --name=kubepro.karampudikarthik.shop \
  --state=s3://kpop-vprofile \
  --zones=us-east-1a,us-east-1b \
  --node-count=2 \
  --node-size=t3.small \
  --master-size=t3.medium \
  --dns-zone=kubepro.karampudikarthik.shop \
  --node-volume-size=8 \
  --master-volume-size=8
```
3. update cluster
```
kops update cluster --name kubepro.karampudikarthik.shop --state=s3://kpop-vprofile --yes --admin
```
4. validate cluster
```
kops validate cluster --state=s3://kpop-vprofile --state=s3://kpop-vprofile 
```

Before writing definition file for pod, we need to create volume for DB pd. so it can be store mysql data in EBS volume

create EBS volume
```
aws ec2 create-volume --volume-type gp2 --size 3 --availability-zone us-east-1a
```
then we will get volume id. copy it and save

when DB pod running on pod. it should be in same zone as volume. we can make that by node selection in definition file. node selector works with LABELS. so we need to create LABEL
```
kubectl label nodes ip-172-3-3-.us-east-1.compute.internal zone=us-east-1a
```
> ip-172-3-3-.us-east-1.compute.internal --- it is node

```
kubectl label nodes ip-172-34-553-.us-east-1.compute.internal zone=us-east-1b
```
when we create cluster. we have 1 master and 2 nodes. here we create LABELS for 2 nodes


















