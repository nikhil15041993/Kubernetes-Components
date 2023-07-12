## Checking the History of the Deployment Rollout

To check the rollout history of a Deployment resource, we can use the history subcommand of the kubectl rollout command. Specifically, we can run kubectl rollout history deployment/myapp-deployment:

```
 kubectl rollout history deployment/myapp-deployment
deployment.apps/myapp-deployment
REVISION  CHANGE-CAUSE
1         <none>
```

The output gives us a Revision column and the Change-Cause column. Whenever there’s a new change to the same Deployment resource, the controller increments the Revision value by one.


Besides that, the Change-Cause column is a column that displays the kubernetes.io/change-cause annotation on our application. The kubernetes.io/change-cause annotation is a way for us to document the changes we apply to the Deployment resource.

```
 kubectl rollout history deployment/myapp-deployment
deployment.apps/myapp-deployment
REVISION  CHANGE-CAUSE
1         <none>
2         update image version from 3.16 to 3.1
```

## Rolling Back a Deployment Rollout
One advantage of having historical records of all the deployment rollouts is that we can roll back our Deployment resources to any previous revisions using the kubectl rollout undo command. For example, we can rollback our myapp-deployment to the immediate previous revision using the kubectl rollout undo command:

```
$ kubectl rollout undo deployment/myapp-deployment
deployment.apps/myapp-deployment rolled back
```


## To roll back to a specific revision, we can pass the –to-revision option followed by the revision number we want to roll back to. For instance, we can rollback the myapp-deployment resource to revision 2 using the –to-revision=2 option:

```
kubectl rollout undo --to-revision=2 deployment/myapp-deployment
```

## Pausing and Resuming a Deployment Rollout

The Deployment controller always monitors and triggers a rollout whenever it detects any specification drift between the running pods and the backing definition. In other words, applying an update to the Deployment resource causes the controller to instantly perform a rolling update to the pods.

To prevent the controller from automatically rolling out the updates, we can issue the kubectl rollout pause command on our Deployment resource. By marking our Deployment resource as paused, the controller won’t automatically roll out the changes on the Deployment level to the pods.

For example, let’s pause our myapp-deployment using kubectl rollout pause:

```
kubectl rollout pause deployment/myapp-deployment
deployment.apps/myapp-deployment paused
```

 ### To check the status of the rollout, we use the kubectl rollout status command, followed by the resource name:


```
kubectl rollout status deployment/myapp-deployment

Waiting for deployment "myapp-deployment" rollout to finish: 0 out of 3 new replicas have been updated...
```

### To resume the rollout, we can issue the kubectl rollout resume command to the resource:
```
 kubectl rollout resume deployment/myapp-deployment
deployment.apps/myapp-deployment resumed
```

### Checking on the rollout status of our myapp-deployment resource now shows that the rollout is successful:
```
$ kubectl rollout status deployment/myapp-deployment
deployment "myapp-deployment" successfully rolled out
```
