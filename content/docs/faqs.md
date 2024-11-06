---
linkTitle: "FAQs"
title: Frequently asked questions
weight: 1000
---

## What is k8sgateway?

{{< reuse "docs/snippets/product-name.md" >}} is an open source, cloud-native Layer 7 proxy that is based on the [Envoy](https://www.envoyproxy.io/). The {{< reuse "docs/snippets/product-name.md" >}} project implements gateway routing by using [{{< reuse "docs/snippets/k8s-gateway-api-name.md" >}}](https://gateway-api.sigs.k8s.io/) resources.

## Why would I want to use k8sgateway?

The {{< reuse "docs/snippets/product-name.md" >}} project was built to support the difficult challenges of monolith to microservice migration, which includes being able to connect multiple types of compute resources, such as virtual machines (VMs) and on-premises monolithic apps with cloud-native, Kubernetes-based apps.

Other use cases {{< reuse "docs/snippets/product-name.md" >}} can solve include the following:

* Kubernetes cluster ingress with a custom {{< reuse "docs/snippets/product-name.md" >}} API as well as native support for the {{< reuse "docs/snippets/k8s-gateway-api-name.md" >}}.
* API gateway functionality for services that run outside Kubernetes

## What’s the difference between k8sgateway and Envoy? 

The Envoy proxy is a data-plane component with powerful routing, observability, and resilience capabilities. However, Envoy can be difficult to operationalize and complex to configure. 

The {{< reuse "docs/snippets/product-name.md" >}} project comes with a simple yet powerful control plane for managing Envoy as an edge ingress, API Gateway, or service proxy. The {{< reuse "docs/snippets/product-name.md" >}} control plane is built on a plugin model that enables extension and customization depending on your environment. This flexibility lets {{< reuse "docs/snippets/product-name.md" >}} adapt both to the fast pace of development in the open source Envoy community, as well as to the unique needs of differing operational environments.

The {{< reuse "docs/snippets/product-name.md" >}} includes the following capabilities beyond open source Envoy:

* A flexible control plane with extensibility in mind
* More ergonomic, domain-specific APIs to drive Envoy configuration
* Function-level routing that goes beyond routing to a `host:port` for clusters, including routing to a Swagger/OpenAPI spec endpoint, gRPC function, cloud provider function such as AWS Lambda, and more
* Transformation of request/response via a super-fast C++ templating filter built on Inja
* Envoy filters to call AWS Lambda directly, handling the complex security handshaking
* Discovery of services running in a hybrid platform such as of virtual machines (VMs), containers, infrastructure as code (IaC), function as a service (FaaS), and so on

## What license is k8sgateway under?

The {{< reuse "docs/snippets/product-name.md" >}} project uses [Apache License 2.0](http://www.apache.org/licenses/).

## What is the project roadmap?

The {{< reuse "docs/snippets/product-name.md" >}} project organizes issues into milestones for release. For more details, see the GitHub project.

## What is the version support policy?

The {{< reuse "docs/snippets/product-name.md" >}} project supports one `n` latest version.

The `main` branch of the `k8sgateway` Git repository is for feature work under development, and is not stable.

## Where is the changelog?

The changelog is part of each [GitHub release](https://github.com/k8sgateway/k8sgateway.io/releases).

## Is there enterprise software that is based on k8sgateway?

{{< cards >}}
  {{< card link="https://www.solo.io/products/gloo-gateway/" title="Solo.io" tag= "Enterprise" image="/img/gloo-gateway-ver-light-on-dark.png" >}}
{{< /cards >}}

## Can I use k8sgateway in a service mesh?

Yes, you can install {{< reuse "docs/snippets/product-name.md" >}} in a service mesh environment, such as Istio.

The {{< reuse "docs/snippets/product-name.md" >}} project is not a service mesh, but can be deployed complementary to a service mesh like Istio. Istio solves the challenges of service-to-service communication by controlling requests as they flow through the system. {{< reuse "docs/snippets/product-name.md" >}} can be deployed at the edge of the service-mesh boundary, between service meshes, or within the mesh to add the following capabilities:

* Mutual TLS (mTLS) encryption of traffic between the gateway and services
* Transformation of request/response to decouple backend APIs from frontend
* Function routing such as AWS Lambda
* Request/response caching
* Unified discovery services of infrastructure like Kubernetes, Consul, Vault, AWS EC2
* Unified discovery services of functions like REST/OAS spec, gRPC reflection, SOAP/WSDL, WebSockets, Cloud Functions, AWS Lambda

For an example, see the [Istio integration guide](/docs/integrations/istio/).

## How do you pronounce k8sgateway?

We say "kates" gateway, although you might commonly hear people say, "Kubernetes gateway."

In the international phonetic alphabet (IPA), this pronunciation is as follows:

```
/keɪts ˈɡeɪtˌweɪ/
```
