# IKS App exposure examples

The following examples deploy an [echo-server][echo-server] to an IBM Kubernetes Cluster and demonstrate how to expose the service to the internet or your IBM Cloud Private network. 

- [x] Classic NLB Version 1 Load Balancer (public)
- [x] Classic NLB Version 2 Load Balancer (public)
- [ ] Classic NLB Load Balancer (private)
- [x] Classic Ingress using TLS and IBM Cloud provided subdomain (public)
- [x] VPC Network Load Balancer (public)
- [ ] VPC Application Load Balancer (private)
- [ ] VPC Ingress using TLS and custom domain (public)

## Choosing an App Exposure Option

Depending on your use case, you may want to use one of the following app exposure options:

| Characteristics | NodePort | Classic NLB | VPC Load Balancer | Ingress |
|---|---|---|---|---|
| Externally accessible | Yes | Yes | Yes  | Yes |
| External hostname | No | Yes  | Yes | Yes |
| Stable external IP | No | Yes | No | Yes |
| HTTP(S) load balancing | No | Yes | Yes | Yes |
| TLS Termination | No | No | No | Yes |
| Custom routing rules | No | No | No | Yes | 
| Multiple apps per service | No | No | No | Yes |

## Deploy the example application

Deploy a simple echo server application to the cluster

```shell
git clone https://github.com/greyhoundforty/iks-app-exposure-examples.git
cd iks-app-exposure-examples
kubectl apply -f echo-server-deploy.yaml
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

**Note**: Update the LoadBalancer annotations to match the zone where you would like the NLB to be deployed. Additionally you can specify the [Scheduling algorithms][nlbv2-annotations] that are used. Scheduling algorithms determine how an NLB 2.0 assigns network connections to your app pods. 

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

```sh
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

```sh
curl --header 'X-ECHO-ENV-BODY: HOSTNAME' https://echo.jb-iks-lab-classic-xxxxxxxxx-0000.us-south.containers.appdomain.cloud

"echoserver-84c789fbd8-4wvnp"
```

[nlbv2-annotations]: https://cloud.ibm.com/docs/containers?topic=containers-loadbalancer-v2#scheduling_supported
[non-default-app]: https://cloud.ibm.com/docs/containers?topic=containers-ingress-types#alb-com-create-ibm-domain
[echo-server]: https://ealenn.github.io/Echo-Server/
