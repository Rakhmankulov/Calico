# Calico change IP pool block size without using another IPPool 

### The official way and our way
Сalico suggests changing ippool by adding a new subnetwork https://docs.tigera.io/calico/latest/networking/ipam/migrate-pools, but we'll do without it

In this example, we will change CIDR `/26` to the new `/28`

### Work Plan
1. We need to change the CIDR in the desired pool
```console
kubectl edit ippool `$NEEDED_POOL_NAME` #once
```

3. Then we need to take the `$NODE_NAME` node off the load
```console
kubectl drain $NODE_NAME --delete-local-data=true --ignore-daemonsets=true
kubectl taint nodes $NODE_NAME key1=value1:NoExecute #if needed
```

3. Then we remove ipamblocks and blockaffinities for the `$NODE_NAME`.
```console
kubectl delete ipamblocks.crd.projectcalico.org $NODE_NAME-POOL
kubectl delete blockaffinities.crd.projectcalico.org $NODE_NAME-$NODE_NAME-POOL
```

4. Untaint and uncordon the `$NODE_NAME`
```console
kubectl taint nodes $NODE_NAME key1=value1:NoExecute-; kubectl uncordon $NODE_NAME
```

6. At the very beginning of the work, redeploy a random, non-dangerous deployment, check that ipamblock is created on the node with the required mask and the pods start successfully.

7. Then, repeat steps 2-4 for all nodes.

### Checking
In the proccess of the work watch the logs of the `Scheduler`, `Kube-controller-manager` and check how the `Pods` are lifted.

### Rollback plan
Absent.
Or do items 1-4, but in the first item roll back `/28` > `/26`.

### Conclusion
The plan proved workable. Everything went smoothly while switching about ten clusters.
