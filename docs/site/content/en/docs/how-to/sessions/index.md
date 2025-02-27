---
title: "Sessions"
description: "Sessions"
lead: ""
date: 2020-10-06T08:48:45+00:00
lastmod: 2020-10-06T08:48:45+00:00
draft: false
images: [ ]
weight: 125
---

## Endpoints

### Start

In order to start a new session - call `POST /api/sessions/start`

```json
{
  "name": "default"
}
```
#### Configure webhooks
You can configure webhooks for a session:
```json
{
  "name": "default",
  "config": {
    "webhooks": [
      {
        "url": "https://httpbin.org/post",
        "events": [
          "message"
        ]
      }
    ]
  }
}
```

Read more about available options on [**Webhooks page ->**]({{< relref "/docs/how-to/webhooks#hmac-authentication" >}})

The configuration is saved and will be applied if the docker container restarts,
and you set `WHATSAPP_RESTART_ALL_SESSIONS` environment variables.
Read more about it in [Autostart section](#autostart).

#### Configure proxy
You can configure proxy for a session by setting `config.proxy` fields:
- `server` - proxy server address, without `http://` or `https://` prefixes
- `username` and `password` - set this if the proxy requires authentication


**No authentication**
```json
{
  "name": "default",
  "config": {
    "proxy": {
      "server": "localhost:3128"
    }
  }
}
```

**Proxy with authentication**
```json
{
  "name": "default",
  "config": {
    "proxy": {
      "server": "localhost:3128",
      "username": "username",
      "password": "P@ssw0rd"
    }
  }
}
```

The configuration is saved and will be applied if the docker container restarts,
and you set `WHATSAPP_RESTART_ALL_SESSIONS` environment variables.
Read more about it in [Autostart section](#autostart).

You can configure proxy when for all sessions by set up environment variables.
Read more about it on [**Configuration page** ->]({{< relref "/docs/how-to/config#proxy" >}}).

### List

To get session list - call `GET /api/sessions`.

The response:

```json
[
  {
    "name": "default",
    "status": "STARTING"
  }
]
```

You can add `?all=true` parameter to the request `GET /api/session?all=True` it'll show you ALL session,
including **STOPPED**,
so you can know which one will be restarted if you set `WHATSAPP_RESTART_ALL_SESSIONS=True` environment variable.

### Stop

In order to stop a new session - call `POST /api/sessions/stop`

{{< alert icon="👉" text="The stop request does not log out the account by default. Set 'logout' field to 'true'." />}}

```json
{
  "name": "default",
  "logout": true
}
```

### Logout

In order to log out the session - call `POST /api/sessions/logout`

{{< alert icon="👉" text="You must stop session first." />}}

```json
{
  "name": "default",
  "logout": true
}
```



## Advanced sessions ![](/images/versions/plus.png)

With [WAHA Plus version]({{< relref "plus-version" >}}) you can save session state to avoid scanning QR code everytime,
configure autostart options so when the docker container restarts - it restores all previously run sessions!

### Session persistent

#### File storage
If you want to save your session and do not scan QR code everytime when you launch WAHA - connect a local file storage
to the container. WAHA stores authentication information in the directory and reuses it after restart.

[Attach volume](https://docs.docker.com/storage/volumes/) part to the command:

```bash
-v `pwd`/.sessions:/app/.sessions
```

The full command would be:

```bash
docker run --rm -d -v `pwd`/.sessions:/app/.sessions -p 3000:3000/tcp --name whatsapp-http-api devlikeapro/whatsapp-http-api-plus
```

#### Remote storage ![](/images/versions/soon.png)

If you're interested in using some "remote" storage (like Redis or other Databases) to save sessions - please create an
issue on GitHub.

For instances, it may be useful if you run WAHA in a cluster of servers and do not have shared file storage

### Autostart
If you don't want to call `POST /api/sessions/start` for every session each time when the container restart -
you can use set of these environment variables to start sessions for you:

- `WHATSAPP_RESTART_ALL_SESSIONS=True`: Set this variable to `True` to start all **STOPPED** sessions after container
  restarts. By default, this variable is set to `False`.
  - Please note that this will start all **STOPPED** sessions, not just the sessions that were working before the restart. You can maintain the session list by
    using `POST /api/session/stop` with the `logout: True` parameter or by calling `POST /api/session/logout` to remove
    **STOPPED** sessions. You can see all sessions, including **STOPPED** sessions, in the `GET /api/sessions/all=True`
    response.
- `WHATSAPP_START_SESSION=session1,session2`: This variable can be used to start sessions with the specified names right
  after launching the API. Separate session names with a comma.


### Multiple sessions

If you want to save server's CPU and Memory - run multiple sessions inside one docker container!
[Plus version]({{< relref "plus-version" >}}) supports multiple sessions in one container.

