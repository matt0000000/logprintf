+++
title = "check file permissions when a container cannot write"
date = "2026-04-07"
description = "volume permission problems can break otherwise valid containers"
tags = [
"docker",
"linux",
"permissions"
]
+++

A container may start correctly but fail when it tries to write to a mounted directory.

Check the directory ownership:

```bash
ls -ld /path/to/data
```

Then compare it with the user running inside the container:

```bash
docker exec container_name id
```

Many mysterious database and configuration errors are simply volume permission problems.

