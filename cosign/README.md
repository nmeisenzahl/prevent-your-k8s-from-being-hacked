# Cosign Demo

## Keyless signing with GitHub Workflows

```bash
kubectl create ns github

kubectl -n github apply -f src/enforce-signed-github.yaml

kubectl -n kyverno logs -f -c kyverno --tail=0 -l "app.kubernetes.io/component=admission-controller,app.kubernetes.io/instance=kyverno"

kubectl -n github run --image=cgr.dev/chainguard/nginx:latest nginx-chainguard-$RANDOM

kubectl -n github run --image=nginx nginx-$RANDOM

kubectl -n github events

kkubectl -n github get pods nginx-chainguard- -ojson | jq -r '.spec.containers[0].image'

DIGEST=$(crane digest cgr.dev/chainguard/nginx:latest) && echo $DIGEST

rekor-cli search --rekor_server https://rekor.sigstore.dev --sha $DIGEST

rekor-cli get --rekor_server https://rekor.sigstore.dev --uuid

crane ls cgr.dev/chainguard/nginx | head
```

## Keyless signing

```bash
kubectl create ns keyless

kubectl -n keyless apply -f src/enforce-signed-keyless.yaml

kubectl -n kyverno logs -f -c kyverno --tail=0 -l "app.kubernetes.io/component=admission-controller,app.kubernetes.io/instance=kyverno"

kubectl -n keyless run --image=ghcr.io/kyverno/test-verify-image:unsigned unsigned-$RANDOM

kubectl -n keyless run --image=ghcr.io/kyverno/test-verify-image:signed-keyless signed-$RANDOM

kubectl -n keylessevents
```

## Verify your own image with local keys (we will reuse our Wolfi image)

```bash
kubectl create ns demo

docker load -i ../wolfi/hello-app.tar

docker push cr0containerconf0demo.azurecr.io/hello-app:latest-amd64

cosign generate-key-pair

cosign sign --key cosign.key $(docker image inspect cr0containerconf0demo.azurecr.io/hello-app:latest-amd64 | jq -r '.[0].RepoDigests[0]')

cosign verify --key cosign.key $(docker image inspect cr0containerconf0demo.azurecr.io/hello-app:latest-amd64

kubectl -n demo apply -f src/enforce-signed.yaml

kubectl -n kyverno logs -f -c kyverno --tail=0 -l "app.kubernetes.io/component=admission-controller,app.kubernetes.io/instance=kyverno"

kubectl -n demo run --image=cr0containerconf0demo.azurecr.io/hello-app:latest-amd64 hello-app-$RANDOM

kubectl -n demo events
```

## Verify your own image with KMS (we will reuse our Wolfi image):

```bash
# Run all the steps from PREPARE.md for this demo

# activate experimental features to enable OCI 1.1 support
export COSIGN_EXPERIMENTAL=1

kubectl create ns demo

docker load -i ../wolfi/hello-app.tar

docker push cr0containerconf0demo.azurecr.io/hello-app:latest-amd64

cosign generate-key-pair \
  --kms azurekms://akv-containerconf-demo.vault.azure.net/containerconf

cosign sign \
  --key azurekms://akv-containerconf-demo.vault.azure.net/containerconf \
  $(docker image inspect cr0containerconf0demo.azurecr.io/hello-app:latest-amd64 | jq -r '.[0].RepoDigests[0]') \
  --registry-referrers-mode oci-1-1

cosign verify \
  --key azurekms://akv-containerconf-demo.vault.azure.net/containerconf \
  cr0containerconf0demo.azurecr.io/hello-app:latest-amd64 | jq

rekor-cli get --rekor_server https://rekor.sigstore.dev --log-index  --format json | jq

oras discover cr0containerconf0demo.azurecr.io/hello-app:latest-amd64 -o tree

oras manifest fetch cr0containerconf0demo.azurecr.io/hello-app@ | jq

cosign public-key \
  --key azurekms://akv-containerconf-demo.vault.azure.net/containerconf \
  && cosign public-key \
  --key azurekms://akv-containerconf-demo.vault.azure.net/containerconf \
  | pbcopy && echo 'Copied public key to clipboard 🔑'

# Copy public key to src/enforce-signed-policy.yaml
kubectl -n demo apply -f src/enforce-signed-policy.yaml

kubectl -n kyverno logs -f -c kyverno --tail=0 -l "app.kubernetes.io/component=admission-controller,app.kubernetes.io/instance=kyverno"

kubectl -n demo run --image=cr0containerconf0demo.azurecr.io/hello-app:latest-amd64 hello-app-$RANDOM

kubectl -n demo events
```
