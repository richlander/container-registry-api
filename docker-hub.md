# GithHub Container Registry

GitHub requires authentication for both public and private images. The scheme for both is a bit different.

## Accessing public images

You can access public images with a no-op token. However, you still need to ask for it, at repo scope.

A token can be requested using the following URL scheme (in C# syntax):

```csharp
string url = $"https://auth.docker.io/token?service=registry.docker.io&scope=repository:{repo}:pull";
```

The following example demonstrates the pattern, requesting a token for the `library/debian` repo. Make sure to quote the string or the `&` will cause `curl` to suspend.

```bash
$ curl "https://auth.docker.io/token?service=registry.docker.io&scope=repository:library/debian:pull"
```

Note: output is quite long, so isn't included.

If you want to make the token easier to use, you can use the following pattern with `jq`:

```bash
$ TOKEN=$(curl -s "https://auth.docker.io/token?service=registry.docker.io&scope=repository:library/debian:pull" | jq -r .token)
$ echo $TOKEN
VERY-LONG-TOKEN
```

After you have the token you can apply it to an `Authorization` header.

```bash
$ curl -s -H "Authorization: Bearer $TOKEN" -H "Accept: application/vnd.docker.distribution.manifest.list.v2+json, application/vnd.docker.distribution.manifest.v2+json" https://index.docker.io/v2/library/debian/manifests/latest
{
  "manifests": [
    {
      "digest": "sha256:9b0e3056b8cd8630271825665a0613cc27829d6a24906dc0122b3b4834312f7d",
      "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
      "platform": {
        "architecture": "amd64",
        "os": "linux"
      },
      "size": 529
    },
    {
      "digest": "sha256:b176e92c806bbf28465bb830a8e2329be9db6b77bc764a337d06b3444d3fa5dc",
      "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
      "platform": {
        "architecture": "arm",
        "os": "linux",
        "variant": "v5"
      },
      "size": 529
    },
    {
      "digest": "sha256:cdd230199482970078c9f0a9914cae8f558c128050a8e2ac60849ebd307f1e45",
      "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
      "platform": {
        "architecture": "arm",
        "os": "linux",
        "variant": "v7"
      },
      "size": 529
    },
    {
      "digest": "sha256:75ba88a40235cbcaa70686e0e63394b435e0a7f4497fa38f88277c4d43f2c384",
      "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
      "platform": {
        "architecture": "arm64",
        "os": "linux",
        "variant": "v8"
      },
      "size": 529
    },
    {
      "digest": "sha256:64ee4b015289e31457dfb6ef3137030b9ba246062cfb71f9d2e7a989d81e3549",
      "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
      "platform": {
        "architecture": "386",
        "os": "linux"
      },
      "size": 529
    },
    {
      "digest": "sha256:6c1349eb961f49f5f43109350a80607bc6c7ac29cd4e1096dea28686839b22d8",
      "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
      "platform": {
        "architecture": "mips64le",
        "os": "linux"
      },
      "size": 529
    },
    {
      "digest": "sha256:70ec4f50ad65c06fab56aaaba32be2aef7b13bb5bc969ad09ae9c78400cc316d",
      "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
      "platform": {
        "architecture": "ppc64le",
        "os": "linux"
      },
      "size": 529
    },
    {
      "digest": "sha256:8ff6cc915cf8a871aa5f6aae2335d9fa43769a5b2a65c72a15605549f3b075a4",
      "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
      "platform": {
        "architecture": "s390x",
        "os": "linux"
      },
      "size": 529
    }
  ],
  "mediaType": "application/vnd.docker.distribution.manifest.list.v2+json",
  "schemaVersion": 2
}
```

## Access private images

The approach to request a private image is similar, but requires a password or PAT. The example will use a PAT.

