# GithHub Container Registry

GitHub requires authentication for both public and private images. The scheme for both is a bit different.

## Accessing public images

You can access public images with a no-op token. However, you still need to ask for it, at repo scope.

A token can be requested using the following URL scheme (in C# syntax):

```csharp
string url = $"https://ghcr.io/token?scope=repository:{repo}:pull";
```

The following example demonstrates the pattern, requesting a token for the `ghcr.io/richlander/dotnet-docker/aspnetapp` repo.

```bash
$ curl https://ghcr.io/token?scope=repository:richlander/dotnet-docker/aspnetapp:pull
{"token":"djE6cmljaGxhbmRlci9kb3RuZXQtZG9ja2VyL2FzcG5ldGFwcDoxNjY1ODgwMTIwMzUyOTY3Mjg0"}
```

If you want to make the token easier to use, you can use the following pattern with `jq`:

```bash
$ TOKEN=$(curl -s https://ghcr.io/token?scope=repository:richlander/dotnet-docker/aspnetapp:pull | jq -r .token)
$ echo $TOKEN
djE6cmljaGxhbmRlci9kb3RuZXQtZG9ja2VyL2FzcG5ldGFwcDoxNjY1ODgwMjQ0MDE4NDMxMTE4
```

After you have the token you can apply it to an `Authorization` header.

```bash
$ curl -s -H "Authorization: Bearer $TOKEN" -H "Accept: application/vnd.docker.distribution.manifest.list.v2+json, application/vnd.docker.distribution.manifest.v2+json" https://ghcr.io/v2/richlander/dotnet-docker/aspnetapp/manifests/main
{
   "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
   "schemaVersion": 2,
   "config": {
      "mediaType": "application/vnd.docker.container.image.v1+json",
      "digest": "sha256:79807ddfc378e5af778da5468303e691f45d5ee0b04c6aeec0f4b96fcdfd39b6",
      "size": 3937
   },
   "layers": [
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "digest": "sha256:bd159e379b3b1bc0134341e4ffdeab5f966ec422ae04818bb69ecef08a823b05",
         "size": 31420102
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "digest": "sha256:e55a07b7e890c54fa94df56be9590be6835718888c746f061dfc526ed2d529ec",
         "size": 14966536
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "digest": "sha256:81957ac33dd22b8cc7db206b57ff1aa08540a8727d70f775509d9a18ff94f6a4",
         "size": 32435440
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "digest": "sha256:eefb39c22ce5dbe64ad685b76b831041c95f6262ad7198f0944aebafa66175ea",
         "size": 156
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "digest": "sha256:fc64b2e24534785dcaf98ff5acf03fcb1e5e261751dfbd2529a92498bc24331a",
         "size": 10110186
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "digest": "sha256:c9c76e30126630a550d8872de0403ca25b6336998e8d33b4d7380e25b554f370",
         "size": 99
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "digest": "sha256:0a43a6cd36d5a5ca1790373cf8bf0d1d478839ad9b55e4ea406cb14c7c2ecd97",
         "size": 1674488
      }
   ]
}
```

## Access private images

The approach to request a private image is similar, but requires a PAT. You only need a single grant.

<img width="557" alt="image" src="https://user-images.githubusercontent.com/2608468/196018877-1717b323-f801-4198-b28e-c0f845916fb9.png">

One you have the PAT, it as to be base64 encoded. You can do that via the following pattern.

```bash
$ PAT=ghp_secret-pat
$ TOKEN=$($PAT | base64)
$ curl -s -H "Authorization: Bearer $TOKEN" -H "Accept: application/vnd.docker.distribution.manifest.list.v2+json, application/vnd.docker.distribution.manifest.v2+json" https://ghcr.io/v2/richlander/dotnet-docker/manifests/main 
{
   "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
   "schemaVersion": 2,
   "config": {
      "mediaType": "application/vnd.docker.container.image.v1+json",
      "digest": "sha256:35c10f6c6124b299afda455b18b4cd5640155a5e5cc36d02f3fe298363786e23",
      "size": 3934
   },
   "layers": [
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "digest": "sha256:31b3f1ad4ce1f369084d0f959813c51df0ca17d9877d5ee88c2db6ff88341430",
         "size": 31404121
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "digest": "sha256:7ed415b4bd19c2b83ef768757b22c5156111db042fd62be4263ba200b4c0c8d0",
         "size": 15171381
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "digest": "sha256:f1df57ed6e1a849b59c3ce9c91e2cea448167c5bdd05dc61dc10576f13a77c13",
         "size": 32412553
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "digest": "sha256:d618ecd62aa0f678dee0bdb66544fd85110c2de1e5c73863cfcdd7aa12691af8",
         "size": 155
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "digest": "sha256:1d834fb288a3b65d0c04ba9c7e02a0fdba5eaac7da348eb97f1de0cca2cac3af",
         "size": 10096969
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "digest": "sha256:251796ca4ad6f9cdc92940bd3eacdd84e894b80f27003e87c39ae587a36f1f53",
         "size": 99
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "digest": "sha256:a72f9a20e811392901fc79dba7f58b0b8f14d6f91cb925ceaf29d9ccb31d56a1",
         "size": 1674402
      }
   ]
}
```
