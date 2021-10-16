---
title: Slow http request from a docker container
tags: python docker
---

Recently I've been having a somewhat mysterious behaviour of docker containers
where a http request from `requests.get` is significantly slower
when it's emitted from a docker container.
When I ran the same command from the host, this slow down does not happen.

```bash
docker run -it requests_test python
```

This simple request below takes 5 seconds
```python
>>> from datetime import datetime
>>> import requests
>>> st = datetime.now(); _ = requests.get('http://www.google.com'); print((datetime.now() - st).total_seconds())
5.065366
```

The docker image `requests_test` is a bare minimum python image like this:
```
FROM python:3.9.7-slim
RUN pip install requests==2.23.0 certifi==2021.5.30
CMD ["python"]
```

I even pinned down the versions of the `requests` package, but it didn't help.

----

It turned out that this slow down was caused by the DNS setting.
My host PC uses a VPN which seems to change the DNS server regularly
while the docker container kept using the old one.

In fact one time the host and docker container had a different DNS configuration
in `/etc/resolv.conf` which is apparently indirectly used by third party applications.

So the solution was to specify a DNS when running `docker.run`.

```bash
> docker run -it --dns xxx.xxx.xxx.xxx requests_test python
```

```python
>>> from datetime import datetime
>>> import requests
>>> st = datetime.now(); _ = requests.get('http://www.google.com'); print((datetime.now() - st).total_seconds())
0.058925
```

Then it came back to normal with 100x speed up.
