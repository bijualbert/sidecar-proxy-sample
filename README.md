# Securing Pods with Sidecar proxies
If an application is deployed as a Pod in a Kubernetes cluster is it by design reachable from every other pod. Which is fine if this application provides strong authentication and authorisation mechanisms. Otherwise other workloads from on Kubernetes cluster can interact with the application without any form of authentication. Even if all workloads from the cluster are trusted and no authentication is required a compromise of any workload can be used as a gateway to this unsecured application. Running a wide range of different workloads in the same cluster with a shared flat network has great benefits but also requires a updated security model. Previously reverse proxies like nginx were often used to encapsulate applications and provide loadbalancing as well as authentication for these applications. Kubernetes provides a simple way to define loadbalancing by configuring Kubernetes Services. But the authentication features that were previously provided by a reverse proxy cannot be provided by Kubernetes Services. Therefore a new approach is needed to provide authentication to applications without their own authentication mechanisms in shared Kubernetes clusters.
One solution for this problem is running a reverse proxy in the front of the application in the same pod as the application as a sidecar container. This is the same model which is used by Istio to provide authentication and authorisation between Service endpoints. Istio injects the Envoy proxy inside Pods. But it is not required to operate a full Servicemesh in order to use the Sidecar proxy pattern. Instead it is possible deploy any proxy software as a sidecar container in the same pod as the application. Because all containers in the same pod share the same virtual network device they can securely interact with each other over the network. This makes it possible to only expose the Sidecare proxy outside the pod and to provide a secure network connection between the proxy and the application. The sidecare proxy will then authenticate any request before forwarding it to the application. In addition to authentication a Sidecar proxy can also provide TLS encryption for applications without TLS support. The proxy can provide a TLS endpoint, terminate the TLS session and forward the non TLS traffic to the application.
Kubernetes Sidecar proxy deployment:
In the picture below the Sidecar proxy pattern is used to provide basic authentication for an application without authentication itself. The proxy has a container port exposed on port 0.0.0.0:80 and is available to other pods. The application container has no container pods exposed and will only listen to requests on the loopback device on port 127.0.01:8080.
 
Sidecar proxy example
All code used in this example is available at https://github.com/eicnix/sidecar-proxy-example
When connecting to the Pod using kubectl port-forward on port 80 it is required to submit the basic authentication credentials to proceed further:
$ kubectl port-forward nginx-deployment-... 8080:80
Forwarding from 127.0.0.1:8080 -> 80
$ curl localhost:8080                                     
<html>
<head><title>401 Authorization Required</title></head>
<body bgcolor="white">
<center><h1>401 Authorization Required</h1></center>
<hr><center>nginx/1.15.1</center>
</body>
</html>
$ curl localhost:8080 -u admin:admin
Hello World!



