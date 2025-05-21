---
title: Backend TLS
weight: 30
description: Configure TLS to terminate for a specific backend workload. 
---

Configure TLS to terminate for a specific backend workload.

When you configure an [HTTPS listener](/docs/setup/listeners/https), the Gateway terminates the TLS connection and decrypts the traffic. The Gateway then routes the decrypted traffic to the backend service.

However, you might have a specific backend workload that uses its own TLS certificate. In this case, you can configure the Gateway to originate a TLS connection that terminates at the backend service by using the {{< reuse "docs/snippets/k8s-gateway-api-name.md" >}} BackendTLSPolicy. For more information, see the [{{< reuse "docs/snippets/k8s-gateway-api-name.md" >}} docs](https://gateway-api.sigs.k8s.io/api-types/backendtlspolicy/).

## Before you begin

{{< reuse "docs/snippets/prereq.md" >}}

## Create a backend workload with a TLS certificate {#workload-tls-cert}

The following example uses an NGINX server with a self-signed TLS certificate. For the configuration, see the [test directory in the kgateway GitHub repository](https://github.com/kgateway-dev/kgateway/tree/{{< reuse "docs/versions/github-branch.md" >}}/test/kubernetes/e2e/features/backendtls/inputs).


1. Deploy the NGINX server with a self-signed TLS certificate.

   ```shell
   kubectl apply -f https://raw.githubusercontent.com/kgateway-dev/kgateway/refs/heads/{{< reuse "docs/versions/github-branch.md" >}}/test/kubernetes/e2e/features/backendtls/inputs/nginx.yaml
   ```

2. Verify that the NGINX server is running.

   ```shell
   kubectl get pods -l app.kubernetes.io/name=nginx
   ```

   Example output:

   ```
   NAME    READY   STATUS    RESTARTS   AGE
   nginx   1/1     Running   0          9s
   ```
   
## Create a BackendTLSPolicy {#create-backend-tls-policy}

Create the BackendTLSPolicy for the NGINX workload. For more information, see the [{{< reuse "docs/snippets/k8s-gateway-api-name.md" >}} docs](https://gateway-api.sigs.k8s.io/api-types/backendtlspolicy/).

1. Install the experimental channel of the {{< reuse "docs/snippets/k8s-gateway-api-name.md" >}} so that you can use BackendTLSPolicy.

   ```shell
   kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v{{< reuse "docs/versions/k8s-gw-version.md" >}}/experimental-install.yaml
   ```

2. Create a Kubernetes ConfigMap that has the public CA certificate for the NGINX server.

   ```shell
   kubectl apply -f- <<EOF
   {{< github url="https://raw.githubusercontent.com/kgateway-dev/kgateway/refs/heads/v2.0.x/test/kubernetes/e2e/features/backendtls/inputs/configmap.yaml" >}}
   EOF
   ```

3. Create the BackendTLSPolicy.

   ```yaml
   kubectl apply -f - <<EOF
   apiVersion: gateway.networking.k8s.io/v1alpha3
   kind: BackendTLSPolicy
   metadata:
     name: nginx-tls-policy
     labels:
       app: nginx
   spec:
     targetRefs:
     - group: ""
       kind: Service
       name: nginx
     validation:
       hostname: "example.com"
       caCertificateRefs:
       - group: ""
         kind: ConfigMap
         name: ca
   EOF
   ```

   {{< reuse "docs/snippets/review-table.md" >}}

   | Setting | Description |
   |---------|-------------|
   | `targetRefs` | The service that you want the Gateway to originate a TLS connection to, such as the NGINX server. |
   | `validation.hostname` | The hostname that matches the NGINX server certificate. |
   | `validation.caCertificateRefs` | The ConfigMap that has the public CA certificate for the NGINX server. |

4. Create an HTTPRoute that routes traffic to the NGINX server on the `example.com` hostname and HTTPS port 8443. Note that the parent Gateway is the sample `http` Gateway resource that you created [before you began](#before-you-begin).

   ```yaml
   kubectl apply -f - <<EOF
   apiVersion: gateway.networking.k8s.io/v1beta1
   kind: HTTPRoute
   metadata:
     name: nginx-route
     labels:
       app: nginx
   spec:
     parentRefs:
     - name: http
       namespace: kgateway-system
     hostnames:
     - "example.com"
     rules:
     - backendRefs:
       - name: nginx
         port: 8443
   EOF
   ```

## Verify the TLS connection {#verify-tls-connection}

Now that your TLS backend and routing resources are configured, verify the TLS connection.

1. Get the external address of the gateway and save it in an environment variable. Note that it might take a few seconds for the gateway address to become available. 
   {{< tabs items="Cloud Provider LoadBalancer,Port-forward for local testing" >}}
   {{% tab %}}
   ```sh
   export INGRESS_GW_ADDRESS=$(kubectl get svc -n kgateway-system http -o jsonpath="{.status.loadBalancer.ingress[0]['hostname','ip']}")
   echo $INGRESS_GW_ADDRESS   
   ```
   {{% /tab %}}
   {{% tab %}}
   ```sh
   kubectl port-forward svc/http -n kgateway-system 8080:8080
   ```
   {{% /tab %}}
   {{< /tabs >}}

2. Send a request to the NGINX server and verify that you get back a 200 HTTP response code. 
   
   {{< tabs items="Cloud Provider LoadBalancer,Port-forward for local testing" >}}
   {{% tab %}}
   ```sh
   curl -vi http://$INGRESS_GW_ADDRESS:8080/ -H "host: example.com:8080"
   ```
   {{% /tab %}}
   {{% tab %}}
   ```sh
   curl -vi http://localhost:8080/ -H "host: example.com:8080"
   ```
   {{% /tab %}}
   {{< /tabs >}}

   Example output: 
   ```
   * Host localhost:8080 was resolved.
   * IPv6: ::1
   * IPv4: 127.0.0.1
   *   Trying [::1]:8080...
   * Connected to localhost (::1) port 8080
   > GET / HTTP/1.1
   > Host: example.com:8080
   > User-Agent: curl/8.7.1
   > Accept: */*
   > 
   * Request completely sent off
   < HTTP/1.1 200 OK
   HTTP/1.1 200 OK
   ```

3. Enable port-forwarding on the Gateway.

   ```sh
   kubectl port-forward deploy/http -n kgateway-system 19000
   ```

4. In your browser, open the Envoy stats page at [http://127.0.0.1:19000/stats](http://127.0.0.1:19000/stats).

5. Search for the following stats that indicate the TLS connection is working. The count increases each time that the Gateway sends a request to the NGINX server.

   * `cluster.kube_default_nginx_8443.ssl.versions.TLSv1.2`: The number of TLSv1.2 connections from the Envoy gateway proxy to the NGINX server.
   * `cluster.kube_default_nginx_8443.ssl.handshake`: The number of successful TLS handshakes between the Envoy gateway proxy and the NGINX server.

## Cleanup

{{< reuse "docs/snippets/cleanup.md" >}}

1. Delete the NGINX server.

   ```yaml
   kubectl delete -f https://raw.githubusercontent.com/kgateway-dev/kgateway/refs/heads/{{< reuse "docs/versions/github-branch.md" >}}/test/kubernetes/e2e/features/backendtls/inputs/nginx.yaml
   ```
   
2. Delete the routing resources that your created for the NGINX server.
   
   ```sh
   kubectl delete backendtlspolicy,configmap,httproute -A -l app=nginx
   ```

3. If you want to re-create a BackendTLSPolicy after deleting one, restart the control plane.

   {{% callout type="warning" %}}
   Due to a [known issue](https://github.com/kgateway-dev/kgateway/issues/11146), if you don't restart the control plane, you might notice requests that fail with a `HTTP/1.1 400 Bad Request` error after creating the new BackendTLSPolicy.
   {{% /callout %}}

   ```sh
   kubectl rollout restart -n kgateway-system deployment/kgateway
   ```
   