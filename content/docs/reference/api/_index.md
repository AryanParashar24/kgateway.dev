---
title: API 
weight: 10
description: API reference
---

Review the API specification for the custom resource definitions that you use in {{< reuse "docs/snippets/product-name.md" >}} .

## k8sgateway API

Use the following links to review the specifications for the {{< reuse "docs/snippets/product-name.md" >}} API: 

{{< cards >}}
  {{< card link="httplisteneroptions" title="HttpListenerOptions" >}}
  {{< card link="routeoptions" title="RouteOptions" >}}
  {{< card link="virtualhostoptions" title="VirtualHostOptions" >}}
{{< /cards >}}

* [Options (including RouteOption, HttpListenerOption, ListenerOption, and VirtualHostOption)](https://docs.solo.io/gloo-edge/main/reference/api/github.com/solo-io/gloo/projects/gloo/api/v1/options.proto.sk/)
* [AuthConfig](https://docs.solo.io/gloo-edge/main/reference/api/github.com/solo-io/gloo/projects/gloo/api/v1/enterprise/options/extauth/v1/extauth.proto.sk/#authconfig)
* [RateLimitConfig](https://docs.solo.io/gloo-edge/main/reference/api/github.com/solo-io/solo-apis/api/rate-limiter/v1alpha1/ratelimit.proto.sk/)
* [Settings](https://docs.solo.io/gloo-edge/main/reference/api/github.com/solo-io/gloo/projects/gloo/api/v1/settings.proto.sk/)
* [Upstream](https://docs.solo.io/gloo-edge/main/reference/api/github.com/solo-io/gloo/projects/gloo/api/v1/upstream.proto.sk/)

## Kubernetes Gateway API

To view configuration options for Gateways, HTTPRoutes, and other resources, see the [{{< reuse "docs/snippets/k8s-gateway-api-name.md" >}} documentation](https://gateway-api.sigs.k8s.io/concepts/api-overview/).


