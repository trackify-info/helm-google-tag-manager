# helm-google-tag-manager - now with Gateway Api Support

## Overview

**Helm Charts for Google Tag Manager** is a [Tag Management System](https://en.wikipedia.org/wiki/Tag_management_system) (TMS) that allows you to quickly and easily update measurement codes and related code fragments, collectively known as **Tags**, on your Website or Mobile App.

Once the small segment of Tag Manager Code has been added to your Project, you can safely and easily deploy Analytics and Measurement Tag Configurations from a Web-Based User Interface.

https://marketingplatform.google.com/about/tag-manager/

## Usage

### Prerequisites

[Helm](https://helm.sh) must be installed first in order to use this Chart.

Please refer to Helm [Documentation](https://helm.sh/docs) to get started.

### Helm Chart Repository

Once Helm has been set up correctly, add locally the Helm Chart Repository as follows:

    helm repo add gtm https://trackify-info.github.io/helm-google-tag-manager

If you had already added this Repository earlier, run `helm repo update` to retrieve the latest versions of this Chart.

You can then run `helm search repo gtm` to see the Chart.

### Install Google Tag Manager Chart

To install the Google Tag Manager Chart:

    helm install gtm gtm/google-tag-manager -f vals.yaml -n ssgtm

### Uninstall Google Tag Manager Chart

To uninstall the Google Tag Manager Chart:

    helm delete gtm -n ssgtm

### Helm Tests

To run Chart Tests:

    helm test gtm -n ssgtm

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

  gateway:
    enabled: true

  ingress:
    enabled: false

    annotations:
      kubernetes.io/ingress.class: "<ingress-class>"
      external-dns.alpha.kubernetes.io/hostname: ps.gtm.trackify.info
      external-dns.alpha.kubernetes.io/ttl: "200"
      cert-manager.io/cluster-issuer: "<cluster-issuer>"
      acme.cert-manager.io/http01-edit-in-place: "true"

    hosts:
      - host: ps.gtm.trackify.info
        paths:
          - path: /
            pathType: ImplementationSpecific

    tls:
      - secretName: ps-gtm-trackify-info-tls
        hosts:
          - ps.gtm.trackify.info

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
  previewServerUrl: https://ps.gtm.trackify.info

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
      external-dns.alpha.kubernetes.io/hostname: sst.gtm.trackify.info
      external-dns.alpha.kubernetes.io/ttl: "200"
      cert-manager.io/cluster-issuer: "<cluster-issuer>"
      acme.cert-manager.io/http01-edit-in-place: "true"

    hosts:
      - host: sst.gtm.trackify.info
        paths:
          - path: /
            pathType: ImplementationSpecific

    tls:
      - secretName: sst-gtm-trackify-info-tls
        hosts:
          - sst.gtm.trackify.info

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
