# Docker Hub Container Registry

Docker Hub requires authentication for both public and private images.

## Accessing public images

You can access public images with a no-op token. However, you still need to ask for it, at repo scope.

A token can be requested using the following URL scheme (in C# syntax):

```csharp
string url = $"https://auth.docker.io/token?service=registry.docker.io&scope=repository:{repo}:pull";
```

The following example demonstrates the pattern, requesting a token for the `library/debian` repo. Make sure to quote the string or the `&` will cause `curl` to suspend.

```bash
$ TOKEN=$(curl -s "https://auth.docker.io/token?service=registry.docker.io&scope=repository:library/debian:pull" | jq -r .token)
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

The approach to request a private image is similar, but requires a username and a password or PAT. The example will use a PAT (see [hub.docker.com](https://hub.docker.com/settings/security)).

To access private images, you'll need a PAT with `Read-only` scope and access to the repo you want to use.

![image](https://user-images.githubusercontent.com/2608468/196059483-ee2db76b-787b-487c-b03b-697e03ee9f69.png)

```bash
$ TOKEN=$(curl -sX POST "https://auth.docker.io/token?service=registry.docker.io&client_id=dockerengine&acc
ess_type=offline" -H "Content-Type: application/x-www-form-urlencoded" -d "grant_type=password&username=${USER}&password=${PAT}&scope=repository:richlander/sdk:pull" | jq -r .access_token)
$ curl -s -H "Accept: application/vnd.docker.distribution.manifest.list.v2+json, application/vnd.docker.distribution.manifest.v2+json" -H "Authorization: Bearer $TOKEN" https://index.docker.io/v2/richlander/sdk/manifests/3.0
{
   "schemaVersion": 2,
   "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
   "config": {
      "mediaType": "application/vnd.docker.container.image.v1+json",
      "size": 6475,
      "digest": "sha256:f5c70f173fbfdcd8c4ac7f6b58a46f3e789468aaf4eb6ee09216e921297e2312"
   },
   "layers": [
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "size": 50380042,
         "digest": "sha256:5ae19949497e04289972756fe51cfac1a72b04fe2709e85a615945035c5a9a61"
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "size": 7804488,
         "digest": "sha256:ed3d96a2798e8837be24597cabf44ce25585cb9db1d749299cb06d51349ea5c2"
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "size": 9978163,
         "digest": "sha256:f1213685078145f6136360475dbaffd0f86dfe92133a7bc26d79602980b255dd"
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "size": 51765388,
         "digest": "sha256:1a9ad5d5550bdff7db4c3d035bf9550bcd1de06a7f178a26de1d082591a5b956"
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "size": 13700784,
         "digest": "sha256:a81c76aa449ef8f338d16c8790d4f2426e5fad6b4a695182caed20cf4084de83"
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "size": 117616338,
         "digest": "sha256:5ebf879c62ec41135cc58be373a19de3653ae41cb72ff009bcdb3adac4505dd9"
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "size": 7168,
         "digest": "sha256:d9162a73644fda790eb0998d706540115e278a2a2a26db730e73f0e4ecdcc4c9"
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "size": 12675895,
         "digest": "sha256:10c85938c7796ea5b4642afdbd91efac7d9cc82ebcfea26f9986a760cbc4f42b"
      }
   ]
}
```

Note: The specific repo use in the example was private at the time of writing. There is no guarantee that it will remain private or even exist. Please use your own private repo. Naturally, you won't have access to my private repos in any case.

## Related resources

- [How to `curl` Docker Registry HTTP API](README.md)
- https://docs.docker.com/registry/spec/auth/
- https://docs.docker.com/docker-hub/access-tokens/
- https://github.com/distribution/distribution/blob/main/docs/spec/auth/token.md
- https://www.arthurkoziel.com/dockerhub-registry-api/
- https://stackoverflow.com/questions/70573806/docker-registry-v2-authentication-using-oauth2-does-not-return-refresh-token-whe
- https://stackoverflow.com/questions/39639129/docker-hub-api-v2-token-authentication-issue
- https://stackoverflow.com/questions/55386202/how-can-i-use-the-docker-registry-api-to-pull-information-about-a-container-get
