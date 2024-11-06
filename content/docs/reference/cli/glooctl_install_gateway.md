---
title: "glooctl install gateway"
description: "Reference for the 'glooctl install gateway' command."
weight: 5
---
## glooctl install gateway

Install {{< reuse "docs/snippets/product-name.md" >}} on Kubernetes.

### Synopsis

requires kubectl to be installed

```
glooctl install gateway [flags]
```

### Options

```
      --create-namespace      Create the namespace to install {{< reuse "docs/snippets/product-name.md" >}} into (default true)
  -d, --dry-run               Dump the raw installation yaml instead of applying it to kubernetes
  -f, --file string           Install {{< reuse "docs/snippets/product-name.md" >}} from this Helm chart archive file rather than from a release
  -h, --help                  help for gateway
  -n, --namespace string      namespace to install {{< reuse "docs/snippets/product-name.md" >}} into (default "gloo-system")
      --release-name string   helm release name (default "gloo")
      --values strings        List of files with value overrides for the {{< reuse "docs/snippets/product-name.md" >}}  Helm chart, (e.g. --values file1,file2 or --values file1 --values file2)
      --version string        version to install (e.g. 1.4.0, defaults to latest)
```

### Options inherited from parent commands

```
  -c, --config string              set the path to the glooctl config file (default "<home_directory>/.gloo/glooctl-config.yaml")
  -i, --interactive                use interactive mode
      --kube-context string        kube context to use when interacting with kubernetes
      --kubeconfig string          kubeconfig to use, if not standard one
  -v, --verbose                    If true, output from kubectl commands will print to stdout/stderr
```

