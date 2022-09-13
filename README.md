# IKS App exposure examples

## Expose Classic IKS Application

### Deploy simple echo server application

```shell
kubectl apply -f classic/echo-server-deployment.yaml
```

### Expose the application via NLB version 1 (non-DSR)

*Note*: Update the LoadBalancer annotations to match the zone where you would like the NLB to be deployed.


```shell
kubectl apply -f classic/classic-nlb-v1.yaml
```

### Expose the application via NLB version 2 (DSR)

*Note*: Update the LoadBalancer annotations to match the zone where you would like the NLB to be deployed. Additionally you can specify the [Scheduling algorithms][nlbv2-annotations] that are used. Scheduling algorithms determine how an NLB 2.0 assigns network connections to your app pods. 

```shell
kubectl apply -f classic/classic-nlb-v1.yaml
```

### Expose the application via Ingress

In order to expose our application via the Ingress controller we need to gather the default `Ingress Subdomain` and `Ingress Secret` (TLS cert) for the cluster. 

```shell
ibmcloud ks cluster get --cluster <cluster_name_or_ID> | grep Ingress
```

**Example output**:

```shell
ibmcloud ks cluster get --cluster $CLASSIC_CLUSTER | grep Ingress
Ingress Subdomain:              jb-iks-lab-classic-xxxxxxxxx-0000.us-south.containers.appdomain.cloud
Ingress Secret:                 jb-iks-lab-classic-xxxxxxxxx-0000
Ingress Status:                 healthy
Ingress Message:                All Ingress components are healthy
```

Update the `classic/classic-ingress.yaml` file with the `Ingress Subdomain` and `Ingress Secret` values from the previous command and then deploy the Ingress resource.

```shell
kubectl apply -f classic/classic-ingress.yaml
```

[nlbv2 annotations]: https://cloud.ibm.com/docs/containers?topic=containers-loadbalancer-v2#scheduling_supported

