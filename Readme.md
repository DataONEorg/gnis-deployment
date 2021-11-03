# GNIS-Deployment

- **License**: [Apache 2](http://opensource.org/licenses/Apache-2.0)
- [**Submit Bugs and feature requests**](https://github.com/DataONEorg/gnis-deployment/issues)


Kubernetes deployment files for the gnis-ld.org project.

## Architecture

The deployment takes place as a single deployment with a single pod, which in turn contains a container for each stack component. For example, there's a container for the triplifier, for the webapp, etc. The webapp is the only pod that communicates with the outside world and has it's port mapping specified in the gnis-service.yaml file. The other containers (triplifier, postgis, GraphDBB) communicate though `localhost` instead of services _because they're in the same pod_.

Because all of the containers exist in the same pod, each container can be accessed from another with `localhost:port`. For example, the webapp gnis-ld can communicate with GraphDB via `localhost:7200`. This is opposed to a service based communication model, which would be appropriate if the containers were in separate pods. This is exploited in the reverse proxy logic in the webapp, where GraphDB is contacted from the webapp.

## Deployment

To deploy the gnis deployment, first check to see if there's a valid PVC. From the `gnis` namespace, run

`kubectl get pvc`

If the PVC exists, there should be a roq entry with State 'Bound'.


If there isn't, create the Persistent Volume (PV) and the Volume Claim (PVC). The PV needs to be created first and with the Kubernetes administrator account in the default namespace. The PVC needs to be created with the `gnis` user in the `gnis` namespace.

Once the PV & PVC exist create the deployment,

`kubectl apply -f deployment/gnis-deployment.yaml`

To create the service to expose the webapp,

`kubectl apply -f networking/gnis-service.yaml`

To perform a rolling update run,

 `kubectl rollout restart deployment gnis`

#### Debugging
There are a few commands that will come in handy while debugging the stack.

Get the logs of a gnis container,

`kubectl logs <pod> <container-name>`

If a container has restarted, the previous logs can be obtained with the `--previous` flag.

`kubectl logs --previous <pod> <container-name>`

To get a shell in a running pod, run

`kubectl exec -i -t <pod> bash`

To get a shell in a container, inside a pod, run

`kubectl exec -n gnis -it pod/<pod-name> -c <container-name> -- /bin/sh`

For debugging networking, the following are useful

`kubectl describe virtualserver -n nginx-ingress`

`kubectl describe virtualserverroute -n gnis`

`kubectl describe service/gnis -n gnis`

An alternative to the rolling update is doing a restart of the deployment. This doesn't delete the PV & PVC

```
kubectl delete deployment gnis
kubectl apply -f templates/deployment/gnis-deployment.yaml
```

##### Debugging GraphDB
In the case that the GraphDB instance needs debugging and access to the GraphDB Workbench is needed, re-configure the gnis service to direct traffic to GraphDB (change `targetPort` to 7200). This will allow you to access the workbench; be sure to change the port back to the gnis-ld port before re-deploying.

[![dataone_footer](https://www.dataone.org/sites/all/images/DataONE_LOGO.jpg)](https://www.dataone.org)