# Wolfi Demo

Build and run the app:

```bash
docker build -t hello:origin -f Dockerfile.origin .

docker run -it --rm -p 8080:5000 hello:origin
```

Let's check for vulnerabilities:

```bash
syft packages --scope all-layers --output syft-json hello:origin > sbom.json

grype sbom:sbom.json

grype --quiet sbom:sbom.json | grep -o deb | wc -l
```

Now build the same app with Wolfi:

```bash
docker build -t hello:wolfi -f Dockerfile.wolfi .

docker run -it --rm -p 8080:5000 hello:wolfi
```

Let's check for OS vulnerabilities again:

```bash
grype cgr.dev/chainguard/node:latest
```

Build with melange and apko:

```bash
melange keygen
melange build --arch amd64,arm64 --signing-key melange.rsa melange.yaml
apko build --arch amd64,arm64 -k melange.rsa.pub apko.yaml cr0containerconf0demo.azurecr.io/hello-app:latest hello-app.tar
```

And finally run it:

```bash
docker load -i hello-app.tar

docker run -it --rm -p 8080:5000 cr0containerconf0demo.azurecr.io/hello-app:latest-arm64
```
