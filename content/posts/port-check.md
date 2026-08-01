+++
title = "find what is using a port on linux"
date = "2026-08-02"
description = "identify the process listening on a specific network port"
tags = [
"linux",
"networking"
]
+++

To find which process is using a port on Linux, run:

```bash
sudo ss -lptn 'sport = :8080'
```

Replace `8080` with the port you want to inspect.

You can also use:

```bash
sudo lsof -i :8080
```

Useful when a service refuses to start because its port is already occupied.

