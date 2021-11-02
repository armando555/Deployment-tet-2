# Process to run the services with kubernetes on GCP
First we have to create a new project on GCP. Later, we have to active and  run the cloud shell. 

We are going to enable the GKE and Cloud SQL


```
gcloud services enable container.googleapis.com sqladmin.googleapis.com
```

Now, we have to configure the environment 
```
gcloud config set compute/zone zone
```
Export a variable with the project id
```
export PROJECT_ID=project-id
```
Later, we have to clone the repo from our GitHub with
```
git clone https://github.com/armando555/Deployment-tet-2.git
```
Go to the directory called wordpress-persistent-disks-kubernetes
```
cd Deployment-tet-2/wordpress-persistent-disks-kubernetes
```
Now, we stablish the work directory with a env variable
```
WORKING_DIR=$(pwd)
``` 

After that, we are going to create the cluster called "persistent-disk-tutorial" with 3 nodes
```
CLUSTER_NAME=persistent-disk-tutorial
gcloud container clusters create $CLUSTER_NAME \
    --num-nodes=3 --enable-autoupgrade --no-enable-basic-auth \
    --no-issue-client-certificate --enable-ip-alias --metadata \
    disable-legacy-endpoints=true
```
Then, we are going to implement the manifest file
``` 
kubectl apply -f $WORKING_DIR/wordpress-volumeclaim.yaml
```
To check it we can run this command
```
kubectl get persistentvolumeclaim
```
You will see something like this
```
NAME                    STATUS    VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
wordpress-volumeclaim   Bound     pvc-89d49350-3c44-11e8-80a6-42010a800002   20G       RWO            standard       5s
```
Now, we are going to implement the manifest file
```
kubectl create -f $WORKING_DIR/wordpress_cloudsql.yaml
```
We can check it with this command
```
kubectl get pod -l app=wordpress --watch
```
It looks like this
```
NAME                     READY     STATUS    RESTARTS   AGE
wordpress-387015-02xxb   2/2       Running   0          9h
```
Now, we are going to create a load balancer service
```
kubectl create -f $WORKING_DIR/wordpress-service.yaml
```
Check it with this command
```
kubectl get svc -l app=wordpress --watch
```
Now we can use the external ip genereted. You can see the external ip with this command
```
NAME        CLUSTER-IP      EXTERNAL-IP    PORT(S)        AGE
wordpress   10.51.243.233   203.0.113.3    80:32418/TCP   1m
```
