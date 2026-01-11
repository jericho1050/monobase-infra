echo "Hello there"
cp -r terraform/examples/aws-eks/ cluster
<!-- change source = "../terraform/modules/aws-eks" in main.tf -->
cd /Users/jerichowenzelserrano/Desktop/monobase-infra/cluster && SSL_CERT_FILE=$(brew --prefix)/etc/openssl@3/cert.pem terraform init 2>&1

cp -r terraform/examples/aws-eks cluster
bun scripts/provision.ts --merge-kubeconfig
bun scripts/provision.ts
bun scripts/provision.ts --merge-kubeconfig
aws eks update-kubeconfig --region ap-southeast-1 --name echo-mycure-staging && kubectl get nodes

gh auth switch

<!-- apply the secrests to mediqueue-staging -->
kubectl create secret generic mongodb \
  --namespace=mediqueue-staging \
  --from-literal=mongodb-root-password="practice-mongo-password-123" \
  --from-literal=mongodb-replica-set-key="practice-replica-key-secret-12345"

<!-- # Create MinIO secret   -->
kubectl create secret generic minio \
  --namespace=mediqueue-staging \
  --from-literal=root-user="minio-admin" \
  --from-literal=root-password="practice-minio-password-123"

bun scripts/bootstrap.ts
bun scripts/bootstrap.ts --skip-github-app

kubectl get pods -n argocd
export | grep "SSL_"
bun scripts/bootstrap.ts --skip-github-app --yes
kubectl get pods
kubectl get ns
kubectl get svc -n gateway-system
kubectl get all -argocd
kubectl get all -n argocd
kubectl port-forward -n argocd service/argocd-server 8080:80
kubectl get svc -n gateway-system
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
kubectl port-forward -n argocd service/argocd-server 8080:80

kubectl get gateway -A

kubectl get svc -A | grep -i loadbalancer


nslookup stg.mediqueue.online

lsd
curl -I http://stg.mediqueue.online
kubectl get pods -n mediqueue-staging

kubectl describe pod -n mediqueue-staging 

kubectl get storageclass

kubectl get pvc -n mediqueue-staging
kubectl patch storageclass gp2 -p '{"metadata": {"annotations":{"storageclass.kubernetes.io/is-default-class":"true"}}}'

kubectl delete pvc datadir-mongodb-0 minio -n mediqueue-staging


kubectl rollout restart statefulset mongodb -n mediqueue-staging
kubectl rollout restart statefulset minio -n mediqueue-staging
kubectl get all -n mediqueue-staging

kubectl logs -n mediqueue-staging mongodb-0

kubectl get pvc -n mediqueue-staging

kubectl describe pod mongodb-0 -n mediqueue-staging | tail -30
kubectl logs -n mediqueue-staging mongodb-0 --previous

kubectl get pods -n mediqueue-staging -w
kubectl get pod mongodb-0 -n mediqueue-staging -o jsonpath='{.status.containerStatuses[0].lastState.terminated}' | jq

kubectl describe pod mongodb-0 -n mediqueue-staging | grep -A5 "Last State"

kubectl top pod mongodb-0 -n mediqueue-staging
kubectl get secret mongodb -n mediqueue-staging -o yaml
kubectl delete statefulset mongodb -n mediqueue-staging

kubectl get pods -n mediqueue-staging -w
kubectl get pods -n mediqueue-staging -w
git add values/deployments/practice-staging.yaml

git commit -m "fix: add MongoDB volume permissions for EKS"

git push

kubectl delete statefulset mongodb -n mediqueue-staging

kubectl delete pvc datadir-mongodb-0 -n mediqueue-staging

kubectl get pods -n mediqueue-staging -w
kubectl get applications -n argocd  # Find the app name


kubectl get pods -n mediqueue-staging -w

kubectl get pod mongodb-0 -n mediqueue-staging -o jsonpath='{.spec.initContainers[*].name}'

kubectl logs -n mediqueue-staging mongodb-0 --all-containers

kubectl get application mediqueue-staging-mongodb -n argocd -o yaml | grep -A20 "status:"
git status

git log
kubectl delete statefulset mongodb -n mediqueue-staging

kubectl delete pvc datadir-mongodb-0 -n mediqueue-staging

kubectl logs -n mediqueue-staging mongodb-0 --all-containers

kubectl annotate application mediqueue-staging-mongodb -n argocd argocd.argoproj.io/refresh=hard --overwrite

kubectl get application mediqueue-staging-mongodb -n argocd -o yaml | grep -A5 volumePermissions

kubectl get pods -n mediqueue-staging -w
kubectl get application mediqueue-staging-mongodb -n argocd -o jsonpath='{.spec.source}' | jq
# Patch the MongoDB application to add volumePermissions
kubectl patch application mediqueue-staging-mongodb -n argocd --type=merge -p '
{
  "spec": {
    "source": {
      "helm": {
        "valuesObject": {
          "arbiter": {"enabled": false},
          "architecture": "replicaset",
          "auth": {"existingSecret": "mongodb", "rootUser": "root"},
          "image": {"repository": "bitnamilegacy/mongodb", "tag": "7.0.15-debian-12-r0"},
          "persistence": {"enabled": true, "size": "10Gi", "storageClass": "gp2"},
          "replicaCount": 1,
          "volumePermissions": {"enabled": true},
          "podSecurityContext": {"enabled": true, "fsGroup": 1001},
          "containerSecurityContext": {"enabled": true, "runAsUser": 1001, "runAsNonRoot": true},
          "resources": {
            "limits": {"cpu": "500m", "memory": "512Mi"},
            "requests": {"cpu": "100m", "memory": "256Mi"}
          }
        }
      }
    }
  }
}'

# Delete the stuck pod to force recreation
kubectl delete pod mongodb-0 -n mediqueue-staging

# Watch
kubectl get pods -n mediqueue-staging -w
kubectl get pods -n mediqueue-staging -w
kubectl get pods -n mediqueue-staging -w
kubectl patch application mediqueue-staging-mongodb -n argocd --type=merge -p '
{
  "spec": {
    "source": {
      "helm": {
        "valuesObject": {
          "architecture": "standalone",
          "auth": {"enabled": false},
          "image": {"repository": "mongo", "tag": "7.0"},
          "persistence": {"enabled": true, "size": "10Gi", "storageClass": "gp2"},
          "resources": {
            "limits": {"cpu": "500m", "memory": "1Gi"},
            "requests": {"cpu": "100m", "memory": "512Mi"}
          }
        }
      }
    }
  }
}'
kubectl delete pod mongodb-0 -n mediqueue-staging

kubectl get pods -n mediqueue-staging -w
kubectl get pods -n mediqueue-staging -w
kubectl get application mediqueue-staging-mongodb -n argocd

kubectl patch application mediqueue-staging-mongodb -n argocd --type merge -p '{"operation":{"initiatedBy":{"username":"admin"},"sync":{}}}'

kubectl get pods -n mediqueue-staging -w

<!-- helm install envoy-gateway oci://docker.io/envoyproxy/gateway-helm --version v1.0.1 -n envoy-gateway-system --create-namespace --skip-crds -->

helm install envoy-gateway oci://docker.io/envoyproxy/gateway-helm --version v1.2.1 -n envoy-gateway-system --skip-crds

