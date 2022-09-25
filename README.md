# Minio Proxy

In my home lab I use TrueNAS Core to run Minio. It exposes a Minio instance that expects SSL but I find configuration of SSL on TrueNAS Core to be a pain. To get around this I created a very simple Nginx based proxy that I can install into Kubernetes. This repository holds the helm chart used to do so.

## Installation

* Clone this repository
* Copy the values.yaml file to your own
* Update the minio and ingress sections, specifying your minio server IP and the ingress values you want. Below is an example of how I configure mine
```
minio:
  host: "192.168.0.16"
  port: "9000"

ingress:
  enabled: true
  className: ""
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-production
    nginx.ingress.kubernetes.io/proxy-body-size: 0m
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: minio-test.dustinrue.com
      paths:
        - path: /
          pathType: ImplementationSpecific
  tls:
   - secretName: minio-test-cert
     hosts:
       - minio-test.dustinrue.com
```
* Apply this to your Kubernetes cluster with `helm install -f settings.yaml -n default minio-proxy .`. Remember to change the commnd to suit your environment.

### Notes

Pay attention to the annotations. You must tell your ingress controller to allow for large body sizes to support large file uploads. The value you may be different depending on your ingress controller. In my cluster I use cert-manager and ingress-nginx for handling SSL certificate generation and ingress.
