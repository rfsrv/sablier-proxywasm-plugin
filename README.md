# sablier-proxywasm-plugin

OCI image wrapper for the [sablierapp/sablier-proxywasm-plugin](https://github.com/sablierapp/sablier-proxywasm-plugin) wasm binary, published to `ghcr.io/rfsrv/sablier-proxywasm-plugin`.

The upstream project publishes the compiled `.wasm` as a GitHub Release asset but does not publish an OCI image. Envoy Gateway's `EnvoyExtensionPolicy` wasm loader requires `type: Image`, so we wrap the binary in a minimal `FROM scratch` image here.

## Releasing

To publish a new version matching an upstream release:

```bash
# Trigger manually via GitHub Actions UI or CLI:
gh workflow run build.yaml -f upstream_version=v1.1.0
```

Or push a tag matching the upstream version:

```bash
git tag v1.1.0
git push origin v1.1.0
```

The workflow will:
1. Download `plugin.wasm` from the upstream GitHub Release
2. Build a `FROM scratch` OCI image containing only `/plugin.wasm`
3. Push `ghcr.io/rfsrv/sablier-proxywasm-plugin:v1.1.0` and `:latest`
4. Attest build provenance

## Image format

```
FROM scratch
COPY plugin.wasm /plugin.wasm
```

Envoy Gateway fetches `/plugin.wasm` from the image layer by convention.

## Pull secret

`ghcr.io` requires authentication even for public packages when pulled by Envoy Gateway's OCI fetcher. A `read:packages` PAT is stored in Doppler as `GHCR_READ_PAT` and materialised as `Secret/ghcr-sablier-wasm` in each app namespace via `ExternalSecret`.
