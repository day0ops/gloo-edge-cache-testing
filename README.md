# JSON Transformation on Gloo Edge

## Prerequisites

1. Create required env vars

    ```
    export CLUSTER_OWNER="kasunt"
    export CLUSTER_REGION=australia-southeast1
    export PROJECT="caching"

    export DOMAIN_NAME=testing.development.internal

    export GLOO_EDGE_HELM_VERSION=1.13.0-beta10
    export GLOO_EDGE_VERSION=v${GLOO_EDGE_HELM_VERSION}
    ```

2. Provisioned cluster
    ```
    ./cluster-provision/scripts/provision-gke-cluster.sh create -n $PROJECT -o $CLUSTER_OWNER -r "${CLUSTER_REGION}"
    ```

## Deployment

### Setup Gloo Edge & Dev Portal

Theres a [pending bug](https://github.com/solo-io/solo-projects/issues/4162) on caching that will prevent from deploying so on a separate terminal
change the `settings` object to use the proper caching upstream (change from `caching-server` to `caching-service`).

```
helm repo add gloo-ee https://storage.googleapis.com/gloo-ee-helm
helm repo update

helm install gloo-ee gloo-ee/gloo-ee -n gloo-system \
  --version ${GLOO_EDGE_HELM_VERSION} \
  --create-namespace \
  --set-string license_key=${GLOO_EDGE_LICENSE_KEY} \
  -f gloo-edge-helm-values.yaml
```

### Setup Application and Configuration

```
kubectl create ns apps
kubectl create ns apps-configuration

kubectl apply -f apps/
kubectl apply -f configuration/
```

## Testing

For testing caching run the following `curl` commands with `max-age` headers.

```
# Retain cache
curl -iv http://<proxy public IP>/service/1/valid-for-minute -H "Cache-Control: max-age=60"
sleep 30
curl -iv http://<proxy public IP>/service/1/valid-for-minute -H "Cache-Control: max-age=60"

# Refresh cache
curl -iv http://<proxy public IP>/service/1/valid-for-minute -H "Cache-Control: max-age=60"
sleep 90
curl -iv http://<proxy public IP>/service/1/valid-for-minute -H "Cache-Control: max-age=60"
```