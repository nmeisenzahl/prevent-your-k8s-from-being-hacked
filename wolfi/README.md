# Wolfi Demo

Build and run golang app:

```bash
nerdctl build -t hello:origin -f Dockerfile.origin .

nerdctl run -it --rm -p 8080:5000 hello:origin
```

Let's check for vulnerabilities:

```bash
nerdctl save hello:origin -o ./hello.tar
syft packages --scope all-layers --output syft-json oci-archive:./hello.tar > sbom.json

grype sbom:sbom.json
```

Now build the same app with Wolfi:

```bash
nerdctl build -t hello:wolfi -f Dockerfile.wolfi .

nerdctl run -it --rm -p 8080:5000 hello:wolfi
```

Let's check for vulnerabilities again:

```bash
cosign download sbom cgr.dev/chainguard/node > sbom.json
grype sbom:sbom.json

nerdctl save hello:wolfi -o ./hello.tar
syft packages --scope all-layers --output syft-json oci-archive:./hello.tar > sbom.json
grype sbom:sbom.json
```

And also compare the image size:

```bash
nerdctl image ls
```

Build with melange and apko:

```bash
nerdctl run --privileged --rm -it -v ${PWD}:/work -w /work --entrypoint /usr/bin/melange cgr.dev/chainguard/sdk:latest keygen
nerdctl run --privileged --rm -it -v ${PWD}:/work -w /work --entrypoint /usr/bin/melange cgr.dev/chainguard/sdk:latest build --arch aarch64 --signing-key melange.rsa --keyring-append melange.rsa melange.yaml
nerdctl run --privileged --rm -it -v ${PWD}:/work -w /work --entrypoint /usr/bin/apko cgr.dev/chainguard/sdk:latest-20230118 build --build-arch aarch64 -k melange.rsa.pub apko.yaml ghcr.io/nmeisenzahl/prevent-your-k8s-from-being-hacked/hello-app:latest hello-app.tar
```
