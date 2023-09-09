# Cosign Demo

Validate a third-party image:

```bash
kubectl apply -f src/enforce-signed-keyless-policy.yaml

kubectl logs -f -n kyverno --tail=0 -l "app.kubernetes.io/name=kyverno"

kubectl run --image=cgr.dev/chainguard/nginx:latest nginx-$RANDOM

kubectl events
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
