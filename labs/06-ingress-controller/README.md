# 6. Implement Ingress Controller with Let's encrypt SSL certificates


# Install cert-manager

1. Prepare namespace

```bash
kubectl create namespace cert-manager
```

2. Disbale validation on namespace


```
kubectl label namespace cert-manager certmanager.k8s.io/disable-validation=true
```

3. Deploy cert-manager

```
kubectl apply -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.7/deploy/manifests/cert-manager.yaml --validate=false
```

## Prepare DNS name

1. Find id of ingress Public IP.

```bash
az network public-ip list --query "[?ipAddress!=null]|[?contains(ipAddress, '<ip>')].[id]" --output tsv
```

2. Set custom helper DNS name for public IP

```bash
az network public-ip update --ids <public ip id> --dns-name <your uniq name>
```

3. Check fqdn for helper DNS name for public IP.

```bash
az network public-ip list --query "[?ipAddress!=null]|[?contains(ipAddress, '<ip>')].[dnsSettings][0][0].[fqdn]" -o tsv
```

## Deploy Let's encrypt SSL certificates with ingress rule

1. Prepare letsencrypt-prod.yaml - put your email.

```yaml
apiVersion: certmanager.k8s.io/v1alpha1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    email: <mail>
    server: https://acme-v02.api.letsencrypt.org/directory
    privateKeySecretRef:
      name: letencrypt-prod-account-key
    # Enable the HTTP01 challenge mechanism for this Issuer
    http01: {}
```

2. Apply letsencrypt-prod.yaml.

```bash
kubectl apply -f letsencrypt-prod.yaml
```

3. Prepare certificates.yaml - put fqdn for helper DNS name for public IP as host.

```yaml
apiVersion: certmanager.k8s.io/v1alpha1
kind: Certificate
metadata:
  name: tls-secret
spec:
  secretName: tls-secret
  dnsNames:
  - <host>
  acme:
    config:
    - http01:
        ingressClass: nginx
      domains:
      - <host>
  issuerRef:
    name: letsencrypt-staging
    kind: ClusterIssuer
```

4. Apply certificates.yaml.

```
kubectl apply -f certificates.yaml
```

5. Prepare ingress.yaml - put fqdn for helper DNS name for public IP as host.

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: hello-world-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    certmanager.k8s.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  tls:
  - hosts:
    - <host>
    secretName: tls-secret
  rules:
  - host: <host>
    http:
      paths:
      - path: /
        backend:
          serviceName: kuard
          servicePort: 80
```

6. Apply ingress.yaml.

```
kubectl apply -f ingress.yaml
```

7. Check web aplication in browser.
