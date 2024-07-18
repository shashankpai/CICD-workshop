#### Canary Rollout 

**Canary Strategies**

##### 1. Rolling Update:

A rolling update is a deployment strategy that gradually replaces old versions of an application with new ones. This ensures that your application remains available to users during the update process, minimizing downtime.

###### Key Terms
Replica: An instance of your application.
Deployment: A configuration that describes how your application should be run, including how many replicas should be running.

###### maxSurge and maxUnavailable
These two settings control the speed and safety of a rolling update by determining how many instances of your application can be created or deleted at a time.

###### maxSurge
What it means: This setting controls how many additional replicas can be created above the desired number of replicas during the update.
Why it's important: It ensures that there are extra replicas running temporarily, which can handle user traffic while some of the old replicas are being replaced by new ones.

###### Example:
If you have a deployment with 5 replicas and set maxSurge: 1, Kubernetes can create up to 6 replicas (5 desired + 1 extra) during the update.

###### maxUnavailable
What it means: This setting controls how many replicas can be taken down (or be unavailable) during the update.
Why it's important: It ensures that there are enough replicas running to handle user traffic, even if some replicas are being updated.

###### Example:

If you have a deployment with 5 replicas and set maxUnavailable: 1, Kubernetes will ensure that at least 4 replicas (5 desired - 1 unavailable) are always running during the update.


     maxSurge
     maxUnavailable
2. SetWeight
3. Traffic-management/traffic-routing
