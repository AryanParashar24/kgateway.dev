---
title: Decode base64 headers
weight: 40
description: Automatically decode base64 values in request headers and add the decoded value as a response header. 
---

In the following example, you combine multiple Inja functions to accomplish the following tasks: 

- Extract a base64-encoded value from a specific request header. 
- Decode the base64-encoded value. 
- Trim the decoded value and only capture everything starting from the 11th character. 
- Add the captured string as a response header. 

## Before you begin

{{< reuse "docs/snippets/prereq.md" >}}

## Decode base64 headers

1. Encode a string to base64. 
   ```sh
   echo -n "transformation test" | base64
   ```
   
   Example output: 
   ```
   dHJhbnNmb3JtYXRpb24gdGVzdA==
   ```
   
2. Create a TrafficPolicy resource with your transformation rules. Make sure to create the TrafficPolicy in the same namespace as the HTTPRoute resource. In the following example, you decode the base64-encoded value from the `x-base64-encoded` request header and populate the decoded value into an `x-base64-decoded` header starting from the 11th character. 

   ```yaml
   kubectl apply -f- <<EOF
   apiVersion: gateway.kgateway.dev/v1alpha1
   kind: TrafficPolicy
   metadata:
     name: transformation
     namespace: httpbin
   spec:
     transformation:
       response:
         set:
         - name: x-base64-decoded
           value: '{{ substring(base64_decode(request_header("x-base64-encoded")), 11) }}'
   EOF
   ```

3. Update the HTTPRoute resource to apply the TrafficPolicy to the httpbin route by using an `extensionRef` filter.

   ```yaml
   kubectl apply -f- <<EOF
   apiVersion: gateway.networking.k8s.io/v1
   kind: HTTPRoute
   metadata:
     name: httpbin
     namespace: httpbin
     labels:
       example: httpbin-route
   spec:
     parentRefs:
       - name: http
         namespace: kgateway-system
     hostnames:
       - "www.example.com"
     rules:
       - backendRefs:
           - name: httpbin
             port: 8000
         filters:
         - type: ExtensionRef
           extensionRef:
             group: gateway.kgateway.dev
             kind: TrafficPolicy
             name: transformation
   EOF
   ```

4. Send a request to the httpbin app and include your base64-encoded string in the `x-base64-encoded` request header. Verify that you get back a 200 HTTP response code and that you see the trimmed decoded value of your base64-encoded string in the `x-base64-decoded` response header. 
   
   {{< tabs items="Cloud Provider LoadBalancer,Port-forward for local testing" >}}
   {{% tab %}}
   ```sh
   curl -vi http://$INGRESS_GW_ADDRESS:8080/response-headers \
    -H "host: www.example.com:8080" \
    -H "x-base64-encoded: dHJhbnNmb3JtYXRpb24gdGVzdA==" 
   ```
   {{% /tab %}}
   {{% tab %}}
   ```sh
   curl -vi localhost:8080/response-headers \
   -H "host: www.example.com" \
   -H "x-base64-encoded: dHJhbnNmb3JtYXRpb24gdGVzdA==" 
   ```
   {{% /tab %}}
   {{< /tabs >}}
   
   Example output: 
   ```yaml {linenos=table,hl_lines=[20,21],linenostart=1}
   * Mark bundle as not supporting multiuse
   < HTTP/1.1 200 OK
   HTTP/1.1 200 OK
   < access-control-allow-credentials: true
   access-control-allow-credentials: true
   < access-control-allow-origin: *
   access-control-allow-origin: *
   < content-type: application/json; encoding=utf-8
   content-type: application/json; encoding=utf-8
   < date: Wed, 26 Jun 2024 02:54:48 GMT
   date: Wed, 26 Jun 2024 02:54:48 GMT
   < content-length: 3
   content-length: 3
   < x-envoy-upstream-service-time: 2
   x-envoy-upstream-service-time: 2
   < server: envoy
   server: envoy
   < x-envoy-decorator-operation: httpbin.httpbin.svc.cluster.local:8000/*
   x-envoy-decorator-operation: httpbin.httpbin.svc.cluster.local:8000/*
   < x-base64-decoded: ion test
   x-base64-decoded: ion test
   ```
   
## Cleanup

{{< reuse "docs/snippets/cleanup.md" >}}

1. Delete the TrafficPolicy resource.

   ```sh
   kubectl delete TrafficPolicy transformation -n httpbin
   ```

2. Remove the `extensionRef` filter from the HTTPRoute resource.

   ```yaml
   kubectl apply -f- <<EOF
   apiVersion: gateway.networking.k8s.io/v1
   kind: HTTPRoute
   metadata:
     name: httpbin
     namespace: httpbin
     labels:
       example: httpbin-route
   spec:
     parentRefs:
       - name: http
         namespace: kgateway-system
     hostnames:
       - "www.example.com"
     rules:
       - backendRefs:
           - name: httpbin
             port: 8000
   EOF
   ```