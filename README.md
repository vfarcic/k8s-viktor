```bash
source secrets/env

K8S_VERSION=[...]

PROJECT=[...]

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
```

## Docker Flow Let's Encrypt

```bash
helm template docker-flow-letsencrypt \
    --name lets-encrypt \
    --output-dir k8s \
    --namespace df

kubectl apply \
    -n df \
    -f k8s/docker-flow-letsencrypt/templates

helm upgrade -i docker-flow-letsencrypt \
    docker-flow-letsencrypt \
    --namespace df
```

## Velero

TODO: Move to environment-viktor-production

```bash
BUCKET=viktor-velero

gsutil mb -p $PROJECT gs://$BUCKET/

gcloud iam service-accounts create velero --project $PROJECT --display-name "Velero service account"

SERVICE_ACCOUNT_EMAIL=$(gcloud iam service-accounts list --project $PROJECT --filter="displayName:Velero service account" --format 'value(email)')

ROLE_PERMISSIONS=(
    compute.disks.get
    compute.disks.create
    compute.disks.createSnapshot
    compute.snapshots.get
    compute.snapshots.create
    compute.snapshots.useReadOnly
    compute.snapshots.delete
    compute.zones.get
)

gcloud iam roles create velero.server \
     --project $PROJECT \
     --title "Velero Server" \
     --permissions "$(IFS=","; echo "${ROLE_PERMISSIONS[*]}")"

gcloud projects add-iam-policy-binding $PROJECT \
    --member serviceAccount:$SERVICE_ACCOUNT_EMAIL \
    --role projects/$PROJECT/roles/velero.server

gsutil iam ch serviceAccount:$SERVICE_ACCOUNT_EMAIL:objectAdmin gs://${BUCKET}

gcloud iam service-accounts keys create credentials-velero \
    --iam-account $SERVICE_ACCOUNT_EMAIL

velero install \
    --provider gcp \
    --bucket $BUCKET \
    --secret-file ./credentials-velero \
    --wait

kubectl logs deployment/velero -n velero

velero schedule create weekly --schedule "0 0 * * 0"

velero schedule get

velero get backups

# BACKUP=[...]

# velero restore create --from-backup $BACKUP --wait
```

NAME               VERSION
jx                 2.0.180
jenkins x platform 2.0.276
Kubernetes cluster v1.13.5-gke.10
kubectl            v1.14.2
helm client        Client: v2.14.0+g05811b8
git                git version 2.20.1 (Apple Git-117)
Operating System   Mac OS X 10.14.5 build 18F203