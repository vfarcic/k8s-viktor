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

source secrets/env

cat prometheus-values.yaml \
    | sed -e "s@SLACK_WEBHOOK_URL@$SLACK_WEBHOOK_URL@g" \
    | tee prometheus-values-secret.yaml

helm upgrade -i prometheus \
    stable/prometheus \
    --namespace metrics \
    --values prometheus-values-secret.yaml

kubectl -n metrics \
    rollout status \
    deployment prometheus-server

helm upgrade -i grafana \
    stable/grafana \
    --namespace metrics \
    --version 1.17.5 \
    --values grafana-values.yml

kubectl -n metrics \
    rollout status \
    deployment grafana

# curl https://raw.githubusercontent.com/solarwinds/fluentd-deployment/master/kubernetes/fluentd-daemonset-papertrail.yaml \
#     | sed -e "s@logsN.papertrailapp.com@$FLUENT_PAPERTRAIL_HOST@g" \
#     | sed -e "s@NNNNN@$FLUENT_PAPERTRAIL_PORT@g" \
#     | sed -e "s@my-cluster-name@$FLUENT_HOSTNAME@g" \
#     | kubectl apply -f -
# 
# kubectl -n kube-system rollout status \
#     ds fluentd-papertrail
```
