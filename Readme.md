# GNIS-Deployment
Kubernetes deployment files for the gnis-ld.org project.


## Deployment

To deploy the gnis pod run,

`kubectl apply -f templates/deployment/gnis-deployment.yaml`

To create the service to expose the webapp run,

`kubectl apply -f templates/service/gnis-service.yaml`

#### Debugging
There are a few commands that will come in handy while debugging the stack.

`kubectl logs <pod> <container-name>`

For example, to get the logs of the 'gnis' container the command is
`kubectl logs <pod> gnis`

If a container has restarted, the previous logs can be obtained with the `--previous` flag.

`kubectl logs --previous <pod> gnis`

To get a shell in a running pod, run

`kubectl exec -i -t <pod> bash`

To get a shell in a container, inside a pod, run

`kubectl exec -n gnis -it pod/<pod-name> -c <container-name> -- /bin/sh`