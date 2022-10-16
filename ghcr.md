# GithHub Container Registry

GitHub requires authentication for both public and private images. The scheme for both is a bit different.

## Accessing public images

You can access public images with a no-op token. However, you still need to ask for it, at repo scope.

A token can be requested using the following URL scheme (in C# syntax):

```csharp
string url = $"https://ghcr.io/token?scope=repository:{repo}:pull";
```

The following example demonstrates the basic pattern, requesting a token for the `ghcr.io/richlander/dotnet-docker/aspnetapp` image.

```bash
$ curl https://ghcr.io/token?scope=repository:richlander/dotnet-docker/aspnetapp:pull
{"token":"djE6cmljaGxhbmRlci9kb3RuZXQtZG9ja2VyL2FzcG5ldGFwcDoxNjY1ODgwMTIwMzUyOTY3Mjg0"}
```

If you want to make the token easier to use, you can use `jq`:

```bash
$ TOKEN=$(curl -s https://ghcr.io/token?scope=repository:richlander/dotnet-docke
r/aspnetapp:pull | jq -r .token)
$ echo $TOKEN
djE6cmljaGxhbmRlci9kb3RuZXQtZG9ja2VyL2FzcG5ldGFwcDoxNjY1ODgwMjQ0MDE4NDMxMTE4
```

## Access private images

The approach to request 