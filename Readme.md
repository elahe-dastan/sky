# sky 
In sky repository I want to put all the stuff I use for learning kubernetes Let me start with this picture

![](image/kubernetese.jpeg)

## Probes
liveness probe: to know when to restart a container<br/>
readiness probe: to know when a container is ready to start accepting traffic<br/>
startup probe: to know when a container application has started<br/>

## Liveness
In the liveness directory I have put a pod configuration and defined a liveness command 

## Deploy mysql
In many cases your app has a database dependency, so you should first deploy a database here I deployed a mysql container
in the stateful directory you can see the files.Let's see what we need, first of all we want a persistent storage like
disk, so our data won't be missed if our pod restarts, so we need a **PersistentVolume** then a **PersistentVolumeClaim**
and after those we can go to the **DeploymentConfig** part and add a volume mount.<br/>
(Note: Many cloud supports(for example the cloud support of the company you are working in) may not let you get persistent 
volumes all you need is a persistent volume claim)

#### strategies
recreate: terminate the old version and release the new one<br/>
ramped: release a new version on a rolling update fashion, one after the other<br/>
blue/green: release a new version alongside the old version then switch traffic<br/>
canary: release a new version to a subset of users, then proceed to a full rollout<br/>
a/b testing: release a new version to a subset of users in a precise way (HTTP headers, cookie, weight, etc.).<br/>