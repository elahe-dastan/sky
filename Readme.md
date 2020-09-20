# sky 
In sky repository I want to put all the stuff I use for learning kubernetes Let me start with this picture

![](image/kubernetese.jpeg)

## Probes
liveness probe: to know when to restart a container<br/>
readiness probe: to know when a container is ready to start accepting traffic<br/>
startup probe: to know when a container application has started<br/>

[comment]:this hasn't been finished

## Deploy mysql
[commeny]:what does selector mean???

#### strategies
recreate: terminate the old version and release the new one<br/>
ramped: release a new version on a rolling update fashion, one after the other<br/>
blue/green: release a new version alongside the old version then switch traffic<br/>
canary: release a new version to a subset of users, then proceed to a full rollout<br/>
a/b testing: release a new version to a subset of users in a precise way (HTTP headers, cookie, weight, etc.).<br/>