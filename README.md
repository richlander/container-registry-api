# How to `curl` Docker Registry HTTP API

Container registries expose a standard api -- [Docker Registry HTTP API V2](https://docs.docker.com/registry/spec/api/) -- that you can call to interact with container images. You can efficiently learn about images by downloading and processing very small image manifests (~2K; JSON format). Calling these APIs via `curl` isn't an alternative to the `docker` CLI, but enables additional workflows.

The following set of commands tell us that nearly 2000 tags have been published to the `dotnet/runtime` repo in the `mcr.microsoft.com` registry.

```bash
curl -s https://mcr.microsoft.com/v2/dotnet/runtime/tags/list | jq .tags | wc -l
1997
```

It's also straightforward to write general [registry clients](https://github.com/mthalman/dredge) or more specific ones that perform tasks like image [up-to-date checks](https://github.com/richlander/lucy).

The remainder of this document discussed image manifests.

## Requesting image by tag

An image manifest can be requested via the following URL scheme (in C# syntax):

```csharp
string url = $"https://{registry}/v2/{repo}/manifests/{tag}";
```

Note: HTTP headers are also required, as you will see.

The following example demonstrates the basic pattern, requesting the manifest for the `mcr.microsoft.com/dotnet/runtime:7.0` image.

```bash
$ curl -s -H "Accept: application/vnd.docker.distribution.manifest.list.v2+json, application/vnd.docker.distribution.manifest.v2+json" https://mcr.microsoft.com/v2/dotnet/runtime/manifests/7.0
{
   "schemaVersion": 2,
   "mediaType": "application/vnd.docker.distribution.manifest.list.v2+json",
   "manifests": [
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 1160,
         "digest": "sha256:344937e3f9ea3f6d04011aa1e8d23270b851dbb5e34eb3a98abb6d90d057d9c5",
         "platform": {
            "architecture": "amd64",
            "os": "linux"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 1160,
         "digest": "sha256:4a8c3ecbc58aeb4906c532f7b61f6a4bf25f120ec0a83c352b6c6a1b5f55d46b",
         "platform": {
            "architecture": "arm",
            "os": "linux",
            "variant": "v7"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 1160,
         "digest": "sha256:134480879f5a964ef89c37c2731ccbd3a31e27b5bbabf4c9821292e01530ebe0",
         "platform": {
            "architecture": "arm64",
            "os": "linux",
            "variant": "v8"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 1754,
         "digest": "sha256:7d5b38b2360e0ec65d91d707b52321101882175969d9446b281acb53a3ffcf51",
         "platform": {
            "architecture": "amd64",
            "os": "windows",
            "os.version": "10.0.17763.3532"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 1755,
         "digest": "sha256:91204424b6c203664d428e526e3c364facfcff29726816fa45a35814b772d564",
         "platform": {
            "architecture": "amd64",
            "os": "windows",
            "os.version": "10.0.20348.1129"
         }
      }
   ]
```

The `mcr.microsoft.com` registry is used since it is one of the few public registries that doesn't require authorization to pull public images. It's also the registry my team uses.

## Manifest schemas

There are multiple versions and types of manifest schema. The latest version is [Image Manifest Version 2, Schema 2](https://docs.docker.com/registry/spec/manifest-v2-2/), which is what will be used throughout this document.

There are multiple media types:

- `application/vnd.docker.distribution.manifest.v1+json`: schema1 (existing manifest format)
- `application/vnd.docker.distribution.manifest.v2+json`: New image manifest format (schemaVersion = 2)
- `application/vnd.docker.distribution.manifest.list.v2+json`: Manifest list, aka “fat manifest”
- `application/vnd.docker.container.image.v1+json`: Container config JSON
- `application/vnd.docker.image.rootfs.diff.tar.gzip`: “Layer”, as a gzipped tar
- `application/vnd.docker.image.rootfs.foreign.diff.tar.gzip`: “Layer”, as a gzipped tar that should never be pushed
- `application/vnd.docker.plugin.v1+json`: Plugin config JSON

The two (`v2`) media types are the key ones. When you walk up to a registry and ask for a modern image, it will either be a container image (`application/vnd.docker.distribution.manifest.v2+json`) or a [multi-arch manifest](https://www.docker.com/blog/tag/multi-architecture/) (`application/vnd.docker.distribution.manifest.list.v2+json`). Unless you know which media type you should be asking for (given the nature of the image), you should ask for both. For example, the `7.0` image above is a `application/vnd.docker.distribution.manifest.list.v2+json` type image. That makes sense since my team wants the `7.0` image to be usable in lots of places.

The example above demonstrates how to request the manifest for a registry tag, w/o a priori knowledge of the image media type. In the case (like we saw) of a manifest list response, then we may want to request information about an actual referenced image. We can now ask for one of the listed `application/vnd.docker.distribution.manifest.v2+json` manifests. We'll target the linux+x64 manifest. It's the same pattern, but we'll use a digest SHA instead of a tag.

```bash
$ curl -s -H "Accept: application/vnd.docker.distribution.manifest.list.v2+json, application/vnd.docker.distribution.manifest.v2+json" https://mcr.microsoft.com/v2/dotnet/runtime/manifests/sha256:344937e3f9ea3f6d04011aa1e8d23270b851dbb5e34eb3a98abb6d90d057d9c5
{
   "schemaVersion": 2,
   "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
   "config": {
      "mediaType": "application/vnd.docker.container.image.v1+json",
      "size": 2836,
      "digest": "sha256:cb1a89e8907e3f9ba2fe0a6d5d131da069bd91a5c2a814004f5a2542b277d827"
   },
   "layers": [
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "size": 31420102,
         "digest": "sha256:bd159e379b3b1bc0134341e4ffdeab5f966ec422ae04818bb69ecef08a823b05"
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "size": 14966536,
         "digest": "sha256:e55a07b7e890c54fa94df56be9590be6835718888c746f061dfc526ed2d529ec"
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "size": 32435440,
         "digest": "sha256:81957ac33dd22b8cc7db206b57ff1aa08540a8727d70f775509d9a18ff94f6a4"
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "size": 156,
         "digest": "sha256:eefb39c22ce5dbe64ad685b76b831041c95f6262ad7198f0944aebafa66175ea"
      }
   ]
}
```

In this image, you see `layers` instead of `manifests` since this we are look at an actual container image. The returned `mediaType` demonstrates that.

This request was made with both the `v2` media types in the `Accept` header. That's not necessary. I didn't bother changing the request to just the `application/vnd.docker.distribution.manifest.v2+json` media type.

You can see that each of the layers are the `application/vnd.docker.image.rootfs.diff.tar.gzip` media type. You can request those in a similar way.

```bash
$ curl -s -H "Accept: application/vnd.docker.image.rootfs.diff.tar.gzip" https://mcr.microsoft.com/v2/dotnet/runtime/blobs/sha256:bd159e379b3b1bc0134341e4ffdeab5f966ec422ae04818bb69ecef08a823b05
<a href="https://westus2.data.mcr.microsoft.com/01031d61e1024861afee5d512651eb9f-h36fskt2ei//docker/registry/v2/blobs/sha256/bd/bd159e379b3b1bc0134341e4ffdeab5f966ec422ae04818bb69ecef08a823b05/data?se=2022-10-16T00%3A09%3A32Z&amp;sig=cTvWiXZD4FZswNFTPXk9U0DrYVrkCwOgc1Do4U79OBI%3D&amp;sp=r&amp;spr=https&amp;sr=b&amp;sv=2016-05-31&amp;regid=01031d61e1024861afee5d512651eb9f">Temporary Redirect</a>.
```

## Authorization

Some registries require authorization, for both public and private content. These use some form of [Docker Registry Token Authentication](https://docs.docker.com/registry/spec/auth/) scheme.

Authorization information is provided by a `Bearer` token in an `Authorization` header.

These schemes are significantly different per registry:

- [Azure Container Registry](acr.md)
- [Docker Hub Registry](docker-hub.md)
- [GitHub Container Registry](ghcr.md)
