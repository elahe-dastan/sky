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
I wanted to deploy a mysql container so I needed 
[comment] persistent volume, pod restart, data reliable, readwriteonce, the second pod access after restart, unique id, ask parham, single instance, strategy = recreate




#### deployment strategies
- recreate: terminate the old version and release the new one
- ramped: release a new version on a rolling update fashion, one after the other
- blue/green: release a new version alongside the old version then switch traffic
- canary: release a new version to a subset of users, then proceed to a full rollout
- a/b testing: release a new version to a subset of users in a precise way (HTTP headers, cookie, weight, etc.).

## Service 
I got a little confused.What does a **Service** exactly do??<br/>
It is both a load balancer and a DNS name.