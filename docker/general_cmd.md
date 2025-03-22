# Why my docker pull failed?
If you’re behind a corporate proxy, Docker might not be able to reach external registries.

```shell
docker info | grep -i proxy
```

If you see HTTP Proxy or HTTPS Proxy, update them in Docker Desktop > Settings > Proxies, or set them manually:

```shell
export HTTP_PROXY="http://your.proxy.com:port"
export HTTPS_PROXY="http://your.proxy.com:port"
```
Then restart Docker and try again.

# how to login with a certain user?
```bash
docker login -u username

# then a prompt will pop up, enter your password
```