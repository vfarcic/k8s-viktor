```bash
source secrets/env

K8S_VERSION=[...]

jx create cluster gke \
    --cluster-name viktor \
    --domain devopstoolkitseries.com \
    --project-id $PROJECT \
    --region us-east1 \
    --machine-type n1-standard-2 \
    --min-num-nodes 1 \
    --max-num-nodes 2 \
    --default-admin-password $JX_PASS \
    --default-environment-prefix viktor \
    --git-provider-kind github \
    --namespace cd \
    --prow \
    --tekton \
    --batch-mode \
    --verbose \
    --kubernetes-version $K8S_VERSION

export LB_IP=$(kubectl -n kube-system get svc jxing-nginx-ingress-controller \
    -o jsonpath="{.status.loadBalancer.ingress[0].ip}")

echo $LB_IP

# Update DNS

kubectl create namespace cert-manager

kubectl label namespace cert-manager certmanager.k8s.io/disable-validation=true

kubectl apply -f https://raw.githubusercontent.com/jetstack/cert-manager/release-0.7/deploy/manifests/cert-manager.yaml

# echo "apiVersion: certmanager.k8s.io/v1alpha1
# kind: ClusterIssuer
# metadata:
#   name: letsencrypt-prod
# spec:
#   acme:
#     server: https://acme-v02.api.letsencrypt.org/directory
#     email: viktor@farcic.com
#     privateKeySecretRef:
#       name: letsencrypt-prod
#     http01: {}" \
#     | kubectl apply -f -

# helm upgrade -i cert-manager \
#     stable/cert-manager \
#     --namespace df \
#     --set ingressShim.defaultIssuerName=letsencrypt-prod \
#     --set ingressShim.defaultIssuerKind=ClusterIssuer

# kubectl get pods

# cat alertmanager.yaml \
#     | sed -e "s@SLACK_WEBHOOK_URL@$SLACK_WEBHOOK_URL@g" \
#     | tee alertmanager-secret.yaml

# kubectl -n cd-production \
#     apply -f alertmanager-secret.yaml
```

TODO: Rewrite

```bash
# TODO: Remove
helm upgrade -i prometheus \
    stable/prometheus \
    --version 8.11.1 \
    --values prometheus-values.yaml \
    --wait
    
helm template docker-flow-letsencrypt \
    --name lets-encrypt \
    --output-dir k8s \
    --namespace df

kubectl apply \
    -n df \
    -f k8s/docker-flow-letsencrypt/templates

kubectl -n metrics \
    rollout status \
    deployment prometheus-server

helm upgrade -i docker-flow-letsencrypt \
    docker-flow-letsencrypt \
    --namespace df

source secrets/env

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

# TODO: Remove nginx Ingress
```
