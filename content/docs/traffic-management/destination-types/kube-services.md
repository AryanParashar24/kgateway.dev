---
title: Kubernetes services
weight: 10
---

Route traffic to a Kubernetes service.

You can route to a Kubernetes service by simply referencing that service in the `backendRefs` section of your HTTPRoute resource as shown in the following example.

{{< callout type="info" >}}
Most guides in this documentation route traffic to a Kubernetes service directly. If you want to configure additional settings for your Kubernetes service, such as the HTTP/2 protocol or a different load balancing algorithm, use an Upstream resource as your backing destination instead. For more information, see [Upstreams](/docs/traffic-management/destination-types/upstreams/). 
{{< /callout >}}

```yaml {linenos=table,hl_lines=[13,14,15],linenostart=1,filename="k8s-service-httproute.yaml"}
apiVersion: gateway.networking.k8s.io/v1beta1
kind: HTTPRoute
metadata:
  name: static-upstream
  namespace: default
spec:
  parentRefs:
  - name: http
    namespace: gloo-system
  hostnames:
    - static.example
  rules:
    - backendRefs:
      - name: httpbin
        port: 8000
      filters:
      - type: ExtensionRef
        extensionRef:
          group: gateway.solo.io
          kind: RouteOption
          name: rewrite
```
