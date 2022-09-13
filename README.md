# IKS App exposure examples

## Expose Classic IKS Application

Deploy a simple echo server application to the cluster:

```shell
kubectl apply -f classic/echo-server-deployment.yaml
```

### Expose the application via NLB version 1 (non-DSR)

**Note**: Update the LoadBalancer annotations to match the zone where you would like the NLB to be deployed.

```shell
kubectl apply -f classic/classic-nlb-v1.yaml
```

#### Test NLB v1 echo server

Retrieve the NLB IP address via `kubectl`

```shell
kubectl get svc echoserver-classic-nlb-v1-service
```

Run curl against NLB IP Address to return the pods `hostnane`:

```shell
curl --header 'X-ECHO-ENV-BODY: HOSTNAME' http://<NLB_IP_ADDRESS>
```

*Example output*

```shell
curl --header 'X-ECHO-ENV-BODY: HOSTNAME' http://192.168.75.29
"echoserver-84c789fbd8-4z7xb"
```

### Expose the application via NLB version 2 (DSR)

*Note*: Update the LoadBalancer annotations to match the zone where you would like the NLB to be deployed. Additionally you can specify the [Scheduling algorithms][nlbv2-annotations] that are used. Scheduling algorithms determine how an NLB 2.0 assigns network connections to your app pods. 

```shell
kubectl apply -f classic/classic-nlb-v1.yaml
```

### Expose the application via Ingress

> If your application is deployed outside of the `default` namespace you will need to follow the directions in [Step 2 here][non-default-app] to copy the `Ingress Secret` to the namespace where the application is deployed.

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

Update the `classic/classic-ingress.yaml` file with the `Ingress Subdomain` and `Ingress Secret` values from the previous command and then deploy the Ingress resource. The application will be exposed at `https://echo.<Ingress Subdomain>`.

```shell
kubectl apply -f classic/classic-ingress.yaml
```

Test connection to TLS subdomain

```shell
curl --header 'X-ECHO-ENV-BODY: HOSTNAME' https://echo.jb-iks-lab-classic-xxxxxxxxx-0000.us-south.containers.appdomain.cloud

"echoserver-84c789fbd8-4wvnp"
```

[nlbv2-annotations]: https://cloud.ibm.com/docs/containers?topic=containers-loadbalancer-v2#scheduling_supported
[non-default-app]: https://cloud.ibm.com/docs/containers?topic=containers-ingress-types#alb-com-create-ibm-domain

