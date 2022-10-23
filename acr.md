# Azure Container Registry

Azure Container Registry is a private registry so requires authorization to access any images.

The following pattern is based off of [Azure Container Registry's support of getting Bearer token using Basic Authentication](https://github.com/Azure/acr/blob/main/docs/Token-BasicAuth.md)

## Create a repository token

You can [Create a token with repository-scoped permissions](https://learn.microsoft.com/azure/container-registry/container-registry-repository-scoped-permissions), which you can then use to access registry APIs. This feature is available in the Premium container registry service tier. There are [other ways of getting a token](https://github.com/Azure/acr/blob/main/docs/AAD-OAuth.md), however this approach is the easiest and possibly has security benefits (the access is scoped).

I created a "pull-only" token via the portal UI for the `aspnetapp` repo in the `richlandertest.azurecr.io` ACR.

![image](https://user-images.githubusercontent.com/2608468/196066431-1225ae9a-e7ef-4d75-b4ed-6346cfde791d.png)


## Acquire a bearer token

We need a bearer token in order to call registry APIs.

```bash
REGISTRY=richlandertest.azurecr.io
REPO=aspnetapp
REPO_USER=readonlypull
REPO_TOKEN=secret-token
BASIC_TOKEN=$(echo -n $REPO_USER:$REPO_TOKEN | base64)
TOKEN=$(curl -sH "Authorization: Basic $BASIC_TOKEN" "https://$REGISTRY/oauth2/token?service=$REGISTRY&scope=repository:$REPO:pull" | jq -r .access_token)
```

## Call registry API

We can now call the registry API using the bearer token.

```bash
curl -s -H "Authorization: Bearer $TOKEN" -H "Accept: application/vnd.docker.distribution.manifest.list.v2+json, application/vnd.docker.distribution.manifest.v2+json" https://$REGISTRY/v2/$REPO/manifests/latest
{
   "schemaVersion": 2,
   "mediaType": "application/vnd.docker.distribution.manifest.v2+json",
   "config": {
      "mediaType": "application/vnd.docker.container.image.v1+json",
      "size": 3924,
      "digest": "sha256:35dfe5982be3fb8b8737b19818df065d265c1fcdc66702fe05a43cd512b47a5c"
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
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "size": 10110186,
         "digest": "sha256:fc64b2e24534785dcaf98ff5acf03fcb1e5e261751dfbd2529a92498bc24331a"
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "size": 90,
         "digest": "sha256:c948b2245b1ee76478f99ac78ce54ae33310d930199ed5b787124f5e47209b66"
      },
      {
         "mediaType": "application/vnd.docker.image.rootfs.diff.tar.gzip",
         "size": 1674041,
         "digest": "sha256:2f6b1fcaca875a200422d983576a91ffbf11750fa26a988e556b5b83e0878160"
      }
   ]
}
```

## Related resources

- [How to `curl` Docker Registry HTTP API](README.md)
- https://github.com/Azure/acr/blob/main/docs/Token-BasicAuth.md
- https://zimmergren.net/using-tokens-and-scope-maps-azure-container-registry-acr-repository-level-access-restriction/
- https://stackoverflow.com/questions/71163896/how-to-authenticate-azure-container-registry-https-registry-azurecr-io-v2
