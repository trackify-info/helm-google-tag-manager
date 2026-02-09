# google-tag-manager

## Overview

**Google Tag Manager** is a [Tag Management System](https://en.wikipedia.org/wiki/Tag_management_system) (TMS) that allows you to quickly and easily update measurement codes and related code fragments, collectively known as **Tags**, on your Website or Mobile App.

Once the small segment of Tag Manager Code has been added to your Project, you can safely and easily deploy Analytics and Measurement Tag Configurations from a Web-Based User Interface.

https://marketingplatform.google.com/about/tag-manager/

## Usage

### Prerequisites

[Helm](https://helm.sh) must be installed first in order to use this Chart.

Please refer to Helm [Documentation](https://helm.sh/docs) to get started.

### Helm Chart Repository

Once Helm has been set up correctly, add locally the Helm Chart Repository as follows:

    helm repo add gtm https://jobtome-labs.github.io/google-tag-manager

If you had already added this Repository earlier, run `helm repo update` to retrieve the latest versions of this Chart.

You can then run `helm search repo gtm` to see the Chart.

### Install Google Tag Manager Chart

To install the Google Tag Manager Chart:

    helm install gtm gtm/google-tag-manager

### Uninstall Google Tag Manager Chart

To uninstall the Google Tag Manager Chart:

    helm delete gtm

### Helm Tests

To run Chart Tests:

    helm test gtm

### Example Values for GKE

The following Example Helm Values are for a **GKE (Google Kubernetes Engine)** Context.

In this context, the [GKE Ingress Controller](https://cloud.google.com/kubernetes-engine/docs/concepts/ingress) is being used, along with [Backend/Frontend Configs](https://cloud.google.com/kubernetes-engine/docs/how-to/ingress-configuration).

```YAML
containerConfig: "<your-container-config>"

previewServer:
  replicaCount: 1

  service:
    type: NodePort

  pdb:
    enabled: true

  ingress:
    enabled: true

    annotations:
      kubernetes.io/ingress.class: "<ingress-class>"
      external-dns.alpha.kubernetes.io/hostname: ps.gtm.jobtome.com
      external-dns.alpha.kubernetes.io/ttl: "200"
      cert-manager.io/cluster-issuer: "<cluster-issuer>"
      acme.cert-manager.io/http01-edit-in-place: "true"

    hosts:
      - host: ps.gtm.jobtome.com
        paths:
          - path: /
            pathType: ImplementationSpecific

    tls:
      - secretName: ps-gtm-jobtome-com-tls
        hosts:
          - ps.gtm.jobtome.com

  frontendConfig:
    enabled: true

    redirectToHttps:
      enabled: true

      responseCodeName: MOVED_PERMANENTLY_DEFAULT

  backendConfig:
    enabled: true

    timeoutSec: 30

    healthCheck:
      checkIntervalSec: 5
      timeoutSec: 3
      healthyThreshold: 1
      unhealthyThreshold: 2
      type: HTTP
      requestPath: /healthz
      port: 8080

serverSideTagging:
  previewServerUrl: https://ps.gtm.jobtome.com

  service:
    type: NodePort

  pdb:
    enabled: true

  autoscaling:
    enabled: false

    minReplicas: 3
    maxReplicas: 10

  ingress:
    enabled: true

    annotations:
      kubernetes.io/ingress.class: "<ingress-class>"
      external-dns.alpha.kubernetes.io/hostname: sst.gtm.jobtome.com
      external-dns.alpha.kubernetes.io/ttl: "200"
      cert-manager.io/cluster-issuer: "<cluster-issuer>"
      acme.cert-manager.io/http01-edit-in-place: "true"

    hosts:
      - host: sst.gtm.jobtome.com
        paths:
          - path: /
            pathType: ImplementationSpecific

    tls:
      - secretName: sst-gtm-jobtome-com-tls
        hosts:
          - sst.gtm.jobtome.com

  frontendConfig:
    enabled: true

    redirectToHttps:
      enabled: true

      responseCodeName: MOVED_PERMANENTLY_DEFAULT

  backendConfig:
    enabled: true

    timeoutSec: 30

    healthCheck:
      checkIntervalSec: 5
      timeoutSec: 3
      healthyThreshold: 1
      unhealthyThreshold: 2
      type: HTTP
      requestPath: /healthz
      port: 8080
```

In addition, you can enable [NEGs](https://cloud.google.com/kubernetes-engine/docs/how-to/standalone-neg).

```YAML
  service:
    neg:
      enabled: true
```


### Example Values for Envoy Gateway

The following Example Helm Values demonstrate how to use **Envoy Gateway** with the **Gateway API** instead of traditional Ingress resources.

This configuration assumes you have [Envoy Gateway](https://gateway.envoyproxy.io/) installed in your cluster and a Gateway resource already created.

```YAML
containerConfig: "<your-container-config>"

previewServer:
  replicaCount: 1

  pdb:
    enabled: true

  gateway:
    enabled: true

    # Reference to your existing Gateway resource
    gatewayName: "eg"
    gatewayNamespace: "envoy-gateway-system"
    
    # Optional: specify a specific listener section
    # sectionName: "https"

    annotations:
      cert-manager.io/cluster-issuer: "<cluster-issuer>"

    hostnames:
      - "ps.gtm.example.com"

    rules:
      - path: /
        pathType: PathPrefix

serverSideTagging:
  previewServerUrl: https://ps.gtm.example.com

  pdb:
    enabled: true

  autoscaling:
    enabled: true
    minReplicas: 3
    maxReplicas: 10

  gateway:
    enabled: true

    # Reference to your existing Gateway resource
    gatewayName: "eg"
    gatewayNamespace: "envoy-gateway-system"

    annotations:
      cert-manager.io/cluster-issuer: "<cluster-issuer>"

    hostnames:
      - "sst.gtm.example.com"

    rules:
      - path: /
        pathType: PathPrefix
```

#### Advanced Gateway Configuration

For more advanced use cases, you can use custom `parentRefs` and filters:

```YAML
previewServer:
  gateway:
    enabled: true
    
    # Custom parentRefs for advanced configurations
    parentRefs:
      - name: eg
        namespace: envoy-gateway-system
        sectionName: https
      - name: eg-internal
        namespace: envoy-gateway-system
        sectionName: http

    hostnames:
      - "ps.gtm.example.com"

    rules:
      - path: /
        pathType: PathPrefix
        # Add custom request headers
        filters:
          - type: RequestHeaderModifier
            requestHeaderModifier:
              add:
                - name: X-Custom-Header
                  value: custom-value
              remove:
                - X-Unwanted-Header
```

#### Using Both Ingress and Gateway

You can enable both Ingress and Gateway simultaneously if needed:

```YAML
previewServer:
  ingress:
    enabled: true
    className: "nginx"
    hosts:
      - host: ps-ingress.gtm.example.com
        paths:
          - path: /
            pathType: Prefix

  gateway:
    enabled: true
    gatewayName: "eg"
    hostnames:
      - "ps-gateway.gtm.example.com"
    rules:
      - path: /
        pathType: PathPrefix
```
