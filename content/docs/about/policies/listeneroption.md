---
title: ListenerOption
weight: 30
description: You can use a ListenerOption resource to attach policies to one, multiple, or all gateway listeners. 
---

You can use a ListenerOption resource to attach policies to one, multiple, or all gateway listeners.

## Policy attachment {#policy-attachment-listeneroption}


Learn more about how you can attach policies to gateway listeners.  

### Option 1: Attach the policy to all listeners on the gateway (`targetRefs`)

You can apply a policy to all the listeners that are defined on the gateway by using the `spec.targetRefs` section in the ListenerOption resource. 

The following ListenerOption resource specifies an access logging policy and applies this policy to a Gateway resource that is named `http`. Because no listener is targeted, the policy applies to all the listeners that are defined on the gateway. 

```console {hl_lines=[7,8,9,10]}
apiVersion: gateway.solo.io/v1
kind: ListenerOption
metadata:
  name: access-logs
  namespace: gloo-system
spec:
  targetRefs:
  - group: gateway.networking.k8s.io
    kind: Gateway
    name: http
  options:
    accessLoggingService:
      accessLog:
      - fileSink:
          path: /dev/stdout
          stringFormat: ""
```

### Option 2: Attach the policy to a particular listener on the gateway (`targetRefs.sectionName`)

Instead of attaching a policy to all the listeners that are defined on the gateway, you can target a particular listener by using the `spec.targetRefs.sectionName` field in the ListenerOption resource. 

The following Gateway resource defines two listeners, an HTTP (`http`) and HTTPS (`https`) listener. 

```console {hl_lines=[8,15]}
kind: Gateway
apiVersion: gateway.networking.k8s.io/v1
metadata:
  name: http
spec:
  gatewayClassName: gloo-gateway
  listeners:
  - name: http
    protocol: HTTP
    port: 8080
    allowedRoutes:
      namespaces:
        from: All
    hostname: www.example.com
  - name: https
    port: 443
    protocol: HTTPS
    hostname: https.example.com
    tls:
      mode: Terminate
      certificateRefs:
        - name: https
          kind: Secret
    allowedRoutes:
      namespaces:
        from: All
```

To apply the policy to only the `https` listener, you specify the listener name in the `spec.targetRefs.sectionName` field in the ListenerOption resource as shown in the following example. 

```console {hl_lines=[11]}
apiVersion: gateway.solo.io/v1
kind: ListenerOption
metadata:
  name: access-logs
  namespace: gloo-system
spec:
  targetRefs:
  - group: gateway.networking.k8s.io
    kind: Gateway
    name: http
    sectionName: https
  options:
    accessLoggingService:
      accessLog:
      - fileSink:
          path: /dev/stdout
          stringFormat: ""
```


## Conflicting policies

If you create multiple ListenerOption resources and attach them to the same gateway listener by using the `targetRefs` option, only the ListenerOption that was first created is applied. 

{{% callout type="info" %}}
You cannot attach multiple ListenerOption resources to the same listener, *even if* they define different top-level policies. To add multiple policies, define them in the same ListenerOption resource.
{{% /callout %}}

In the following image, you want to attach two ListenerOption resources to the HTTP listener. One adds an access logging policy and the other one defines connection buffer limits. Because only one ListenerOption can be attached to a gateway listener via `targetRefs` at any given time, only the policy that is created first is enforced (policy 1). 

{{< reuse-image src="img/policy-ov-multiple-listeneroption.svg" width="800" >}}