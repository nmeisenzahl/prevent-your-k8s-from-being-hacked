# Cosign Demo

Validate a third-party image:

```bash
kubectl apply -f src/enforce-signed-keyless-policy.yaml

kubectl logs -f -n kyverno --tail=0 -l "app.kubernetes.io/name=kyverno"

kubectl run --image=cgr.dev/chainguard/nginx:latest nginx-$RANDOM

kubectl events

rekor-cli search --rekor_server https://rekor.sigstore.dev  --sha sha256:9619cdefed8f52a964f3e522a936b1f02f0bc8bc6eff2589bf330ec2c70ea346

rekor-cli get --rekor_server https://rekor.sigstore.dev --uuid 24296fb24b8ad77a795683764476a3cdc94c639e888c5ff2b4fcaa8094cab3e546ba28ee83d6131c

cosign verify cgr.dev/chainguard/nginx:latest \
  --certificate-identity=https://github.com/chainguard-images/images/.github/workflows/release.yaml@refs/heads/main \
  --certificate-oidc-issuer=https://token.actions.githubusercontent.com

```

Verify your own image (we will reuse our Wolfi image):

```bash
nerdctl load -i ../wolfi/hello-app.tar

nerdctl push ghcr.io/nmeisenzahl/prevent-your-k8s-from-being-hacked/hello-app:latest

cosign generate-key-pair

cosign sign --key cosign.key $(nerdctl image inspect ghcr.io/nmeisenzahl/prevent-your-k8s-from-being-hacked/hello-app:latest | jq -r '.[0].RepoDigests[0]')

kubectl apply -f src/enforce-signed-policy.yaml

kubectl logs -f -n kyverno --tail=0 -l "app.kubernetes.io/name=kyverno"

kubectl run --image=ghcr.io/nmeisenzahl/prevent-your-k8s-from-being-hacked/hello-app:latest hello-app-$RANDOM

kubectl events
```
