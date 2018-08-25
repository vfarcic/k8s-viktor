```bash
echo "apiVersion: certmanager.k8s.io/v1alpha1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: viktor@farcic.com
    privateKeySecretRef:
      name: letsencrypt-prod
    http01: {}" \
    | kubectl apply -f -

helm upgrade -i cert-manager \
    stable/cert-manager \
    --namespace df \
    --set ingressShim.defaultIssuerName=letsencrypt-prod \
    --set ingressShim.defaultIssuerKind=ClusterIssuer

helm upgrade -i jenkins jenkins \
    --namespace cd

helm upgrade -i docker-flow-letsencrypt \
    docker-flow-letsencrypt \
    --namespace df

helm upgrade -i prometheus \
    stable/prometheus \
    --namespace metrics \
    --values prometheus-values.yaml

kubectl -n metrics \
    rollout status \
    deploy prometheus-server
```