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
         "digest": "sha256:2dfb924c56bcd4686574471abdb42134a53e9179947b1c598639f56d97161fdd",
         "platform": {
            "architecture": "amd64",
            "os": "linux"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 1160,
         "digest": "sha256:213e687504ec1f6050e30609e054b975331c483cea5d15f804a6d6374f89f4ae",
         "platform": {
            "architecture": "arm",
            "os": "linux",
            "variant": "v7"
         }
      },
      {
         "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
         "size": 1160,
         "digest": "sha256:1fc87d6c1f375360d67e1f1df55703e72c2962929b082769d35d5174d9d98ae1",
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
}
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

The example above demonstrates how to request the manifest for a registry tag, w/o a priori knowledge of the image media type. In the case (like we saw) of a manifest list response, then we may want to request information about an actual referenced image. We can now ask for one of the listed `application/vnd.docker.distribution.manifest.v2+json` manifests. We'll target the linux+x64 manifest. It's the same pattern, but we'll use a digest SHA instead of a tag. It's from the `curl` response above.

```bash
$ curl -s -H "Accept: application/vnd.docker.distribution.manifest.list.v2+json, application/vnd.docker.distribution.manifest.v2+json" https://mcr.microsoft.com/v2/dotnet/runtime/manifests/sha256:2dfb924c56bcd4686574471abdb42134a53e9179947b1c598639f56d97161fdd
{
   "schemaVersion": 2,
   "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
   "config": {
      "mediaType": "application/vnd.docker.container.image.v1+json",
      "size": 2836,
      "digest": "sha256:0aeb78cec714e0e0062824f19ed6ff1b6747fececf864f0951b2b446bb4eb3fd"
   },
   "layers": [
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "size": 31420038,
         "digest": "sha256:e9995326b091af7b3ce352fad4d76cf3a3cb62b7a0c35cc5f625e8e649d23c50"
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "size": 14967219,
         "digest": "sha256:83280acc7c2c9fb3fecc49c3a89d133b323d7b79565f4b483014ed6f15e490b3"
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "size": 32435439,
         "digest": "sha256:b08fd3a9929c3fadd980f0d51915837fe299657c3d62f2fc04bab3646961415b"
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "size": 155,
         "digest": "sha256:058f65a20fb5ccb9c513d000e1a8ef5b1eefa936edf2503afb391dd9233c3034"
      }
   ]
}
```

In this image, you see `layers` instead of `manifests` since this we are look at an actual container image. The returned `mediaType` demonstrates that.

This request was made with both the `v2` media types in the `Accept` header. That's not necessary. I didn't bother changing the request to just the `application/vnd.docker.distribution.manifest.v2+json` media type.

You can see that each of the layers are the `application/vnd.docker.image.rootfs.diff.tar.gzip` media type. You can request those in a similar way.

```bash
$ curl -s -H "Accept: application/vnd.docker.image.rootfs.diff.tar.gzip" https://mcr.microsoft.com/v2/dotnet/runtime/blobs/sha256:e9995326b091af7b3ce352fad4d76cf3a3cb62b7a0c35cc5f625e8e649d23c50
<a href="https://westus2.data.mcr.microsoft.com/01031d61e1024861afee5d512651eb9f-h36fskt2ei//docker/registry/v2/blobs/sha256/e9/e9995326b091af7b3ce352fad4d76cf3a3cb62b7a0c35cc5f625e8e649d23c50/data?se=2022-10-30T22%3A08%3A45Z&amp;sig=FOpRRXh5FdQ1dALu3qWKh8XarsZEeiDqvpatwz4cfGc%3D&amp;sp=r&amp;spr=https&amp;sr=b&amp;sv=2016-05-31&amp;regid=01031d61e1024861afee5d512651eb9f">Temporary Redirect</a>.
```

If you request that content, you will see that `curl` warns you that the content is binary data. There is a reason for that (it's binary data).

```bash
$ curl "https://westus2.data.mcr.microsoft.com/01031d61e1024861afee5d512651eb9f-h36fskt2ei//docker/registry/v2/blobs/sha256/e9/e9995326b091af7b3ce352fad4d76cf3a3cb62b7a0c35cc5f625e8e649d23c50/data?se=2022-10-30T22%3A08%3A45Z&amp;sig=FOpRRXh5FdQ1dALu3qWKh8XarsZEeiDqvpatwz4cfGc%3D&amp;sp=r&amp;spr=https&amp;sr=b&amp;sv=2016-05-31&amp;regid=01031d61e1024861afee5d512651eb9f"
Warning: Binary output can mess up your terminal. Use "--output -" to tell
Warning: curl to output it to your terminal anyway, or consider "--output
Warning: <FILE>" to save to a file.
```

However, the first digest in the manifest (before the `layers` content) is special. It's not binary data, but metadata for the image. Let's take a look.

```bash
$ curl -s -H "Accept: application/vnd.docker.image.rootfs.diff.tar.gzip" https://mcr.microsoft.com/v2/dotnet/runtime/blobs/sha256:0aeb78cec714e0e0062824f19ed6ff1b6747fececf864f0951b2b446bb4eb3fd
<a href="https://westus2.data.mcr.microsoft.com/01031d61e1024861afee5d512651eb9f-h36fskt2ei//docker/registry/v2/blobs/sha256/0a/0aeb78cec714e0e0062824f19ed6ff1b6747fececf864f0951b2b446bb4eb3fd/data?se=2022-10-30T22%3A06%3A37Z&amp;sig=DX72eZm4uMgOO9WTHY%2BxJ%2BdDXHglRlGmNv5qqjSVadg%3D&amp;sp=r&amp;spr=https&amp;sr=b&amp;sv=2016-05-31&amp;regid=01031d61e1024861afee5d512651eb9f">Temporary Redirect</a>.
$ curl -s "https://westus2.data.mcr.microsoft.com/01031d61e1024861afee5d512651eb9f-h36fskt2ei//docker/registry/v2/blobs/sha256/0a/0aeb78cec714e0e0062824f19ed6ff1b6747fececf864f0951b2b446bb4eb3fd/data?se=2022-10-30T22%3A06%3A37Z&amp;sig=DX72eZm4uMgOO9WTHY%2BxJ%2BdDXHglRlGmNv5qqjSVadg%3D&amp;sp=r&amp;spr=https&amp;sr=b&amp;sv=2016-05-31&amp;regid=01031d61e1024861afee5d512651eb9f" | jq
{
  "architecture": "amd64",
  "config": {
    "Hostname": "",
    "Domainname": "",
    "User": "",
    "AttachStdin": false,
    "AttachStdout": false,
    "AttachStderr": false,
    "Tty": false,
    "OpenStdin": false,
    "StdinOnce": false,
    "Env": [
      "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
      "ASPNETCORE_URLS=http://+:80",
      "DOTNET_RUNNING_IN_CONTAINER=true",
      "DOTNET_VERSION=7.0.0-rc.2.22472.3"
    ],
    "Cmd": [
      "bash"
    ],
    "Image": "sha256:0b8d1fd16e32475264e397313f6ef8cee32621b5ed2238053542169f2ea1e70d",
    "Volumes": null,
    "WorkingDir": "",
    "Entrypoint": null,
    "OnBuild": null,
    "Labels": null
  },
  "container": "99f45221908cd63d1a250ee16beed4bb4862a1107aca7d607c89e3114d02c65f",
  "container_config": {
    "Hostname": "",
    "Domainname": "",
    "User": "",
    "AttachStdin": false,
    "AttachStdout": false,
    "AttachStderr": false,
    "Tty": false,
    "OpenStdin": false,
    "StdinOnce": false,
    "Env": [
      "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin",
      "ASPNETCORE_URLS=http://+:80",
      "DOTNET_RUNNING_IN_CONTAINER=true",
      "DOTNET_VERSION=7.0.0-rc.2.22472.3"
    ],
    "Cmd": [
      "/bin/sh",
      "-c",
      "ln -s /usr/share/dotnet/dotnet /usr/bin/dotnet"
    ],
    "Image": "sha256:0b8d1fd16e32475264e397313f6ef8cee32621b5ed2238053542169f2ea1e70d",
    "Volumes": null,
    "WorkingDir": "",
    "Entrypoint": null,
    "OnBuild": null,
    "Labels": null
  },
  "created": "2022-10-25T04:26:19.445847737Z",
  "docker_version": "20.10.17",
  "history": [
    {
      "created": "2022-10-25T01:43:53.171278246Z",
      "created_by": "/bin/sh -c #(nop) ADD file:8644a8156a07a656a35c41e2b2a458befb660309f8592e3efd5b43d46156cec2 in / "
    },
    {
      "created": "2022-10-25T01:43:53.514250664Z",
      "created_by": "/bin/sh -c #(nop)  CMD [\"bash\"]",
      "empty_layer": true
    },
    {
      "created": "2022-10-25T04:25:57.83090842Z",
      "created_by": "/bin/sh -c apt-get update     && apt-get install -y --no-install-recommends         ca-certificates                 libc6         libgcc1         libgssapi-krb5-2         libicu67         libssl1.1         libstdc++6         zlib1g     && rm -rf /var/lib/apt/lists/*"
    },
    {
      "created": "2022-10-25T04:25:58.281747234Z",
      "created_by": "/bin/sh -c #(nop)  ENV ASPNETCORE_URLS=http://+:80 DOTNET_RUNNING_IN_CONTAINER=true",
      "empty_layer": true
    },
    {
      "created": "2022-10-25T04:26:16.267058345Z",
      "created_by": "/bin/sh -c #(nop)  ENV DOTNET_VERSION=7.0.0-rc.2.22472.3",
      "empty_layer": true
    },
    {
      "created": "2022-10-25T04:26:18.544328798Z",
      "created_by": "/bin/sh -c #(nop) COPY dir:f0df7b7df9c2211bc690d2eb1e7cd5aa5ad3773275d0f2a818a43b148923e2c9 in /usr/share/dotnet "
    },
    {
      "created": "2022-10-25T04:26:19.445847737Z",
      "created_by": "/bin/sh -c ln -s /usr/share/dotnet/dotnet /usr/bin/dotnet"
    }
  ],
  "os": "linux",
  "rootfs": {
    "type": "layers",
    "diff_ids": [
      "sha256:a12586ed027fafddcddcc63b31671f406c25e43342479fc92a330e7e30d65f2e",
      "sha256:55125ebb892069bd16e8dedc05e382be6a818ed99026f2e5f8578852b7c6d1cb",
      "sha256:7a10df1adcc9c0f6ec933d9502afd8bdd4c9d058559f63315cb7008e2b5c2222",
      "sha256:b866408b8d940b8b100d176939c5a01cdffdb17792e310329dca7f3d6d41c7b5"
    ]
  }
}
```

This looks just like `docker inspect`. Yes.

## Authorization

Some registries require authorization, for both public and private content. These use some form of [Docker Registry Token Authentication](https://docs.docker.com/registry/spec/auth/) scheme.

Authorization information is provided by a `Bearer` token in an `Authorization` header.

These schemes are significantly different per registry:

- [Azure Container Registry](acr.md)
- [Docker Hub Registry](docker-hub.md)
- [GitHub Container Registry](ghcr.md)
