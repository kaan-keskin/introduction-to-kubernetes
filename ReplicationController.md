## ReplicationController

In real-world use cases, you want your deployments to stay up and running automatically and remain healthy without any manual intervention. To do this, you almost never create pods directly. Instead, you create other types of resources to create and manage the actual pods.

A ReplicationController is a Kubernetes resource that ensures its pods are always kept running. If the pod disappears for any reason, such as in the event of a node disappearing from the cluster or because the pod was evicted from the node, the ReplicationController notices the missing pod and creates a replacement pod.

<img src=".\images\p3_replicationcontroller_createpod.jpg"/>

A ReplicationController constantly monitors the list of running pods and makes sure the actual number of pods of a “type” always matches the desired number. If too few such pods are running, it creates new replicas from a pod template. If too many such pods are running, it removes the excess replicas.

Replication-Controllers operate on sets of pods that match a certain label selector.

<img src=".\images\p3_replicationcontroller_reconciliation_loop.jpg"/>

ReplicationController provides the following features:

* It makes sure a pod (or multiple pod replicas) is always running by starting a new pod when an existing one goes missing.
* When a cluster node fails, it creates replacement replicas for the pods that were under the Replication-Controller’s control and running on the failed node.
* It enables horizontal scaling of pods—both manual and automatic.

    > A pod instance is never relocated to another node. Instead, the Replication-Controller creates a completely new pod instance that has no relation to the instance it’s replacing.

### Creating a ReplicationController

ReplicationController is created by posting a JSON or YAML descriptor to the Kubernetes API server.

<img src=".\images\p3_replicationcontroller_yaml_example.jpg"/>

> Don’t specify a pod selector when defining a ReplicationController. Let Kubernetes extract it from the pod template. This will keep your YAML shorter and simpler.

### Responding Pod/Node Failures

The controller responds to the deletion of a pod by creating a new replacement pod. Technically, it isn’t responding to the deletion itself, but the resulting state—the inadequate number of pods.

In the non-Kubernetes world, If a node fails, the ops team would need to migrate the applications running on that node to other machines manually. Kubernetes, on the other hand, does that automatically. Soon after the ReplicationController detects that its pods are down, it will spin up new pods to replace them.

### Moving pods in and out of the scope of a ReplicationController

ReplicationController manages pods that match its label selector. By changing a pod’s labels, it can be removed from or added to the scope of a ReplicationController. It can even be moved from one ReplicationController to another.

You need to either remove matching label(s) or change its value to move the pod out of the ReplicationController’s scope. Adding another label will have no effect, because the ReplicationController doesn’t care if the pod has any additional labels. It only cares whether the pod has all the labels referenced in the label selector.

<img src=".\images\p3_replicationcontroller_change_pod_label.jpg"/>

> Removing a pod from the scope of the ReplicationController comes in handy when you want to perform actions on a specific pod. For example, you might have a bug that causes your pod to start behaving badly after a specific amount of time or a specific event. If you know a pod is malfunctioning, you can take it out of the ReplicationController’s scope, let the controller replace it with a new one, and then debug or play with the pod in any way you want. Once you’re done, you delete the pod.

> What happens if we change label-selector of ReplicationController? 
> 
> It would make all the pods fall out of the scope of the ReplicationController and create three new pods.

> What happends if we change pod template?
>
> A ReplicationController’s pod template can be modified at any time. Changing the pod template only affect the pods created afterwards. To modify the old pods, you’d need to delete them and let the Replication-Controller replace them with new ones based on the new template.

<img src=".\images\p3_replicationcontroller_change_pod_template.jpg"/>

### Horizontally scaling pods
Scaling the number of pods up or down is executed by changing the value of the replicas field in the ReplicationController resource. After the change, the Replication-Controller will either delete some existing pods (when scaling down) or create additional pods (when scaling up).

* Edit yaml file and apply new configuration: kubectl apply -f replication-set.yaml

* kubectl scale rc kubia --replicas=6

### Deleting a ReplicationController

When you delete a ReplicationController through kubectl delete, the pods are also deleted. 

 pods created by a ReplicationController aren’t an integral part of the ReplicationController, and are only managed by it, you can delete only the ReplicationController and leave the pods running by using --cascade=false flag.