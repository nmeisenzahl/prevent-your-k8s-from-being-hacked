# Cosign Demo

Keyless signing with GitHub Workflows:

```bash
kubectl apply -f src/enforce-signed-github.yaml

kubectl logs -f -n kyverno --tail=0 -l "app.kubernetes.io/name=kyverno"

kubectl run --image=cgr.dev/chainguard/nginx:latest nginx-$RANDOM

kubectl events

crane digest cgr.dev/chainguard/nginx:latest

rekor-cli search --rekor_server https://rekor.sigstore.dev  --sha $DIGEST

rekor-cli get --rekor_server https://rekor.sigstore.dev --uuid 

crane ls cgr.dev/chainguard/nginx | head
```

Keyless signing:

```bash
kubectl apply -f src/enforce-signed-keyless.yaml

kubectl logs -f -n kyverno --tail=0 -l "app.kubernetes.io/name=kyverno"

kubectl run --image=ghcr.io/kyverno/test-verify-image:unsigned unsigned-$RANDOM

kubectl run --image=ghcr.io/kyverno/test-verify-image:signed-keyless signed-$RANDOM

kubectl events
```

Verify your own image with local keys (we will reuse our Wolfi image):

```bash
nerdctl load -i ../wolfi/hello-app.tar

nerdctl push ghcr.io/nmeisenzahl/prevent-your-k8s-from-being-hacked/hello-app:latest

cosign generate-key-pair

cosign sign --key cosign.key $(nerdctl image inspect ghcr.io/nmeisenzahl/prevent-your-k8s-from-being-hacked/hello-app:latest | jq -r '.[0].RepoDigests[0]')

kubectl apply -f src/enforce-signed.yaml

kubectl logs -f -n kyverno --tail=0 -l "app.kubernetes.io/name=kyverno"

kubectl run --image=ghcr.io/nmeisenzahl/prevent-your-k8s-from-being-hacked/hello-app:latest hello-app-$RANDOM

kubectl events
```

Verify your own image with KMS (we will reuse our Wolfi image):

```bash
# Run all the steps from PREPARE.md for this demo

# activate experimental features to enable OCI 1.1 support
export COSIGN_EXPERIMENTAL=1

nerdctl load -i ../wolfi/hello-app.tar

nerdctl push cr0containerconf0demo.azurecr.io/hello-app:latest

cosign generate-key-pair \
  --kms azurekms://akv-containerconf-demo.vault.azure.net/signtest

cosign sign \
  --key azurekms://akv-containerconf-demo.vault.azure.net/signtest \
  $(nerdctl image inspect cr0containerconf0demo.azurecr.io/hello-app:latest | jq -r '.[0].RepoDigests[0]') \
  --registry-referrers-mode oci-1-1

cosign verify \
  --key azurekms://akv-containerconf-demo.vault.azure.net/signtest \
  cr0containerconf0demo.azurecr.io/hello-app:latest | jq

rekor-cli get --rekor_server https://rekor.sigstore.dev --log-index  --format json | jq

oras discover cr0containerconf0demo.azurecr.io/hello-app:latest -o tree

oras manifest fetch cr0containerconf0demo.azurecr.io/hello-app@ | jq

cosign public-key \
  --key azurekms://akv-containerconf-demo.vault.azure.net/signtest

# Copy public key to src/enforce-signed-policy.yaml
kubectl apply -f src/enforce-signed-policy.yaml

kubectl logs -f -n kyverno --tail=0 -l "app.kubernetes.io/name=kyverno"

kubectl run --image=cr0containerconf0demo.azurecr.io/hello-app:latest hello-app-$RANDOM

kubectl events
```
