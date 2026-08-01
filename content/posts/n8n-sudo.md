+++
title = "sudo, n8n and the missing terminal"
date = "2026-07-17"
description = "Getting smartctl to run over n8n's SSH node"
tags = [
"n8n",
"linux",
"selfhosted"
]
+++
I have an n8n workflow that SSHes into my server and runs smartctl to check drive health. It kept failing with "sudo: a terminal is required to read the password".

The SSH node opens a non-interactive session with no TTY, so sudo has no way to prompt me. The fix is to stop it asking at all: a passwordless rule in /etc/sudoers.d, scoped to smartctl only rather than opening up sudo entirely.

That still didn't work, because sudoers matches on the exact command path you write in the rule. On Debian /usr/sbin isn't in a normal user's PATH, only root's, so `which smartctl` comes back empty and it's easy to write the rule against a path that doesn't resolve. Use /usr/sbin/smartctl in both the sudoers rule and the script and it goes through.
