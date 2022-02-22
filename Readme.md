# sky 
In sky repository I want to put all the stuff I use for learning kubernetes Let me start with this picture

<p align="center">
  <img src="image/kubernetese.jpeg"></img>
</p>

## Probes
- liveness probe: to know when to restart a container
- readiness probe: to know when a container is ready to start accepting traffic
- startup probe: to know when a container application has started

## Liveness
In the liveness directory, I have put a pod configuration and defined a liveness command.

## Deploy mysql
In many cases your app has a database dependency, so you should first deploy a database.
Here I deployed a mysql container that in the stateful directory you can see the files.
Let's see what we need, first of all we want a persistent storage like disk, so our data won't be missed if our pod restarts, so we need a `PersistentVolume` then a `PersistentVolumeClaim` and after those we can go to the `DeploymentConfig` part and add a volume mount.

(Note: Many private cloud providers (for example the private cloud of the company you are working in) may not let you get persistent volumes so all you need is a persistent volume claim)

## Statefulsets
I want to tell the story I went through, one day my team lead asked to deploy a project on the cloud with it's dependencies,
I had no experience in working with kubernetese so I started from very beginning I thought that I had to deploy the 
project's database before deploying itself I found a very simple sample (stateful directory) and everything was fine till
I asked myself a question what does `clusterIP: None` mean??? well, to be honest I didn't completely understand what it
means :sweat_smile: but it led me to another concept `statefulsets` I want to explain it as complete as I can.<br/>

#### Workloads
Workloads are objects that set deployment rules for pods.Based on these rules, Kubernetese performs the deployment and
updates the workload with the current state of the application.Workloads let you define the rules for application
scheduling, scaling and upgrade.The most frequently used workloads are `Deployment` and `StatefulSets`. A question probably
popped up in your mind is this what is the difference??? They say `Deployment` should be used for stateless applications
and `StatefulSets` should be used for stateful applications, now we come to this question Why?? Now I want to continue my
story I wanted to deploy a database, so my application was stateful and if I wanted to listen to their advice I had to use
`StatefulSets` workload, but the example I was following had used `Deploymeny` workload which confused me.Here is what was
going on.<br/>
I wanted to deploy a mysql container, so I needed a `PersistentVolume`, the pod may restart for various reasons and if I 
don't use an underlying persistent volume I lose all my data.I have put the access mode of the persistent volume equal to
`ReadWriteOne` it means that the storage can only be accessed by a single node **but wait** what if the pod restarts??!
**then the newer pod cannot access the persistent volume cause it's another pod??!** here is why we use Statefulsets.
##### StatefulSets VS Deployment
When we use `Deployment` workload if a pod restarts then the newer pod is completely different from the first one, and the
persistent volume thinks it's another node and doesn't let access, but statefulsets workload manage pods with their unique
id (for example hostname) so if a pod restarts the newer pod has the same unique id, so the persistent volume thinks it's
the same node as a result the pod has all the failed pod accesses.Another question rises up, in my stateful directory I 
have used deployment workload so what if it restarts, it cannot connect to the persistent volume any more??! well, what I
have done in that directory is called `Single-instance stateful application`.

##### Single-instance stateful application
if I manage more than one pod then you're right everything will be broken but in that specific example everything works 
fine because<br/>
- first:It is single instance
- second:it's `Strategy` is `Recreate`

I cannot have more than one pod, **Why?** the underlying persistent volume is ReadWriteOnce so more than one pod cannot 
access it.You may ask about what the statefulset workload does, **it gives each pod its own persistent volume**.<br/>
The strategy should be `Recreate` so the first pod terminates completely and a new one with the exact same configuration 
will be managed, so it doesn't loose any access.

With the preceding explanation you should have concluded that in case of a stateful application `Rolling Update` is impossible.

##### Accessing mysql instance
I had my mysql pod up and running, I decided to have a mysql client pod and use it query my mysql. I wrote this command:<br/>
```sh
$ kubectl run -it --rm --image=mysql:8.0 --restart=Never --requests=cpu=100m,memory=256Mi mysql-client -- mysql -h mysql -ppassword
```
and unexpectedly I got this error<br/>
failed to open log file "/var/log/pods/...":open /var/log/pods/... no such file or directory<br/>
This error can have so many reasons, in my case it needed a persistent volume. **Why??** read another story:<br/>
We all know that everything in a container lives as long as the container is up and running and since the container be
deleted all the data in the container is gone forever except we manage a volume. Once, I wanted to create a mysql 
container and I hadn't passed any volume I deleted the container and after some time I created another mysql container
surprisingly all the previous data was there :flushed: it happens because in the `Dockerfile` of mysql image is set a 
volume.
I was using `OpenShift` and openshift checks that the dockerfile needs a volume and doesn't let me create a pod without
it.

#### deployment strategies
- recreate: terminate the old version and release the new one
- ramped: release a new version on a rolling update fashion, one after the other
- blue/green: release a new version alongside the old version then switch traffic
- canary: release a new version to a subset of users, then proceed to a full rollout
- a/b testing: release a new version to a subset of users in a precise way (HTTP headers, cookie, weight, etc.).

## Service 
I got a little confused.What does a **Service** exactly do??<br/>
It is both a load balancer and a DNS name.

## Empty Dir
I want to create a redis pod and I have the same problem that I had while creating mysql pod.Redis has a volume mount in 
it's Dockerfile but I don't want the data on my redis to be persistent in fact I want my redis to be as fast as possible,
so I don't want it to write data on the disk as a result I used Empty Dir. Here is more explanation:
An emptyDir volume is first created when a Pod is assigned to a Node, and exists as long as that Pod is running on that 
node. As the name says, it is initially empty. Containers in the Pod can all read and write the same files in the emptyDir 
volume, though that volume can be mounted at the same or different paths in each Container. When a Pod is removed from a 
node for any reason, the data in the emptyDir is deleted forever.(Note: A Container crashing does NOT remove a Pod from 
a node, so the data in an emptyDir volume is safe across Container crashes.)

## Disable Redis Persistence
Redis cloud provides two methods to persist the data to disk. They are Append Only File (AOF) and Snapshot (RDB).
Snapshot or RDB takes the data at that moment and writes it to the disk, in a binary file called dump.rdb.
On the other hand, AOF writes all the incoming “write” commands to the disk. Data persistence is optional, and we can disable it too.
Though persistence prevents data loss, it increases the disk space usage abundantly. So, in cases where we can't persist 
large data on disk, we disable persistence.

## Route
In order for services to be exposed externally, an OpenShift route allows you to associate a service with an 
externally-reachable host name. This edge host name is then used to route traffic to the service. 

## lost + found
Once, I wanted to create a postgres pod, and I got an error containing the key word `lost + found`, here is the problem,
when you create a pod or a container and set a **volume** it is mandatory that the directory on the host that you wanna 
mount to be completely "empty" which is not always the case. A simple solution is to use `PGDATA`, this is what happens 
you have a `mount path` which is a path inside your container and this path is mounted to the root of a disk as a volume, 
but the root of the disk is not empty, so you don't change the mount path, and it is still mounted to the root of the disk, 
but you say that the `PGDATA` which is the place on the disk that you want to write and read from is a subdirectory on the 
disk.

## Common oc commands
1. push manifest to kubernetes: oc apply -f <file>
2. BuildConfig manifest is a kind to build an image and we run it by: oc start-build <name>
3. you can get your buildconfigs by: oc get bc
## docker container ip
## oc vs kubectl

## If the container doesn't have anything to do?? busybox-sleep
## invalid port range
## Connect to one pod from another
