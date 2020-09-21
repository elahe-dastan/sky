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

#### deployment strategies
- recreate: terminate the old version and release the new one
- ramped: release a new version on a rolling update fashion, one after the other
- blue/green: release a new version alongside the old version then switch traffic
- canary: release a new version to a subset of users, then proceed to a full rollout
- a/b testing: release a new version to a subset of users in a precise way (HTTP headers, cookie, weight, etc.).
