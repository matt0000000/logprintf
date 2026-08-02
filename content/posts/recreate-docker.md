+++
title = "recreate a container after configuration changes"
date = "2026-04-23"
description = "restarting a container may not apply updated settings"
tags = [
"docker",
"selfhosted",
"troubleshooting"
]
+++

I changed a container's environment variables and restarted it, but nothing changed.

A restart only stops and starts the existing container. It does not necessarily rebuild it with the new configuration.

Recreate the service instead:

```bash
docker compose up -d --force-recreate service_name
```

When configuration changes are ignored, make sure the container was actually recreated.

