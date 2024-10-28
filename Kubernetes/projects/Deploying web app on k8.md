## Deploying web app on k8

project link :https://github.com/devopshydclub/vprofile-project

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


create a git repo(kube-app). ther we keep all the k8 definition file
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

it will create voulme in aws account. go to aws > EBS > click on volume > add tag
> key- `KubernetesCluster` , value - `kubepro.karampudikarthik.shop` # cluster name

when DB pod running on pod. it should be in same zone as volume. we can make that by node selection in definition file. node selector works with LABELS. so we need to create LABEL
```
kubectl label nodes ip-172-3-3-.us-east-1.compute.internal zone=us-east-1a
```
> ip-172-3-3-.us-east-1.compute.internal --- it is node

```
kubectl label nodes ip-172-34-553-.us-east-1.compute.internal zone=us-east-1b
```
when we create cluster. we have 1 master and 2 nodes. here we create LABELS for 2 nodes

already we have 2 image in docker hub
1. vprofiledb
2. vprofileapp

### we need to encrypt the passwords
we cannot put the passwords in string format, we need to encrytpt it
```
encho -n "vprodbpass" | base64
```
> vprodbpass is the password

```
encho -n "guest" | base64
```
> guest is rabbitmq password

now, we get encrypt code. copy and save it. we need to paste it in app-secret.yaml file


open intellij idea and create a app-secret.yaml file on git repo
```
apiVersion: v1
kind: Secret
metadata: 
    name: app-secret
type: Opaque
data:
    db-pass: dbjfibbf==
    rmq-pass: dcjnc=
```

save and clone it to ec2 and go to kube-app folder. there we have our app-secret.yaml
```
kubectl create -f app-secret.yaml
```
```
kubectl get secret
```
```
kubectl describe secret
```
we can see the detailed info

## DB definition file
create a file vprodbdep.yml

we need to refer to application.properties file

```
apiVersion: apps/v1
kind: Deployment
metadata: 
    name: vprodb
    labels:
    app: vprodb
spec:
    selector:
        matchLabels:
            app: vprodb
    replicas: 1
    template:
        metadata:
          labels:
            app: vprodb
        spec:
            containers:
                - name: vprodb
                  image: krazy666777/vprofiledb:V1
                  volumeMounts:
                    - mountPath: /var/lib/mysql
                      name: vpro-db-data
                  ports:
                    - name: vprodb-port
                      containerPort: 3306
                  env:
                    - name: MYSQL_ROOT_PASSWORD
                      valueFrom:
                        secretKeyRef:
                            name: app-secret
                            key: db-pass
            nodeSelector:
                zone: us-east-1a
            volumes:
                - name: vpro-db-data
                  awsElasticBlockStore:
                    volumeID: vol-nfbui84fhnij
                    fsType: ext4

            initContainers:
                - name: busybox
                  image: busybox:latest
                  args: ["rm", "-rf", "/var/lib/mysql/lost+found"]
                  volumeMounts:
                    - mountPath: /var/lib/mysql
                      name: vpro-db-data

```

this label going to be our service difinition file. so we gng to a service of type cluster  IP and our service is gng to route the request to any pod that have the label `vprodb`

commit and push to git

pull it to ec2 and test it
```
kubectl create -f vprodbdep.yaml
```
to check
```
kubectl get deploy
```
```
kubectl get pod
```
```
kubectl describe pod vprodb-fmdfhjdiw-svblt
```

### create service for db
create a file db-CIP.yaml
> this file is only for internal traffic

```
apiVersion: v1
kind: Service
metadata: 
    name: vprodb
spec:
    ports:
        - port: 3306
          targetPort: vprodb-port  # it will see in deployment file
          protocol: TCP
    selector:
        app: vprodb
```

### deployment file memcache
create a file mcdep.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata: 
    name: vpromc
    labels:
    app: vpromc
spec:
    selector:
        matchLabels:
            app: vpromc
    replicas: 1
    template:
        metadata:
          labels:
            app: vpromc
        spec:
            containers:
                - name: vpromc
                  image: memcached
                  ports:
                    - name: vpromc-port
                      containerPort: 11211
```

now create a service definition file for this `mc-CIP.yaml`







