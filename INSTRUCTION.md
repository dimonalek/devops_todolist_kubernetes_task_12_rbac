# Validation Instructions

## Prerequisites

- kind
- kubectl
- curl

## 1. Create the Kubernetes cluster

```bash
kind create cluster --config cluster.yml
```

## 2. Deploy all application resources

From the repository root, run:

```bash
chmod +x bootstrap.sh
./bootstrap.sh
```

If you are using PowerShell on Windows, run the applies one by one:

```powershell
kubectl apply -f .infrastructure/mysql/ns.yml
kubectl apply -f .infrastructure/mysql/configMap.yml
kubectl apply -f .infrastructure/mysql/secret.yml
kubectl apply -f .infrastructure/mysql/service.yml
kubectl apply -f .infrastructure/mysql/statefulSet.yml

kubectl apply -f .infrastructure/app/ns.yml
kubectl apply -f .infrastructure/app/pv.yml
kubectl apply -f .infrastructure/app/pvc.yml
kubectl apply -f .infrastructure/app/secret.yml
kubectl apply -f .infrastructure/app/configMap.yml
kubectl apply -f security/rbac
kubectl apply -f .infrastructure/app/clusterIp.yml
kubectl apply -f .infrastructure/app/nodeport.yml
kubectl apply -f .infrastructure/app/hpa.yml
kubectl apply -f .infrastructure/app/deployment.yml
```

## 3. Verify ServiceAccount is used by Deployment

```bash
kubectl -n todoapp get deploy todoapp -o jsonpath='{.spec.template.spec.serviceAccountName}'
```

Expected output:

```text
todoapp-secrets-reader
```

## 4. Verify RBAC objects exist

```bash
kubectl -n todoapp get sa todoapp-secrets-reader
kubectl -n todoapp get role todoapp-secrets-reader
kubectl -n todoapp get rolebinding todoapp-secrets-reader
```

## 5. Execute curl from the Deployment pod to list secrets

Get a pod name:

```bash
kubectl -n todoapp get pods -l app=todoapp
```

Run the curl command from inside the pod:

```bash
kubectl -n todoapp exec deploy/todoapp -- sh -c 'TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token); CACERT=/var/run/secrets/kubernetes.io/serviceaccount/ca.crt; curl -sS --cacert $CACERT -H "Authorization: Bearer $TOKEN" https://kubernetes.default.svc/api/v1/namespaces/todoapp/secrets'
```

Expected result:

- Response contains a JSON payload with `"kind":"SecretList"`
- Response includes `items` with secrets from namespace `todoapp`

## 6. Take screenshot for PR

- Capture a screenshot of the terminal output from step 5.
- Attach the screenshot to the PR.

## 7. Cleanup (optional)

```bash
kind delete cluster
```
