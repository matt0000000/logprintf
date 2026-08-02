+++
title = "nohup is enough"
date = "2026-05-08"
description = "Running a long migration over SSH without tmux"
tags = [
"linux",
"ssh",
"selfhosted"
]
+++
Starting a multi-hour rsync over SSH, I reached for tmux out of habit and got a socket error instead of a session.

I could have fixed the socket, but the requirement was smaller than the tool. All I actually needed was for the process to survive my connection dropping, which is one line: `nohup rsync ... > /tmp/migration.log 2>&1 &`, then `tail -f` on the log whenever I want to check in.

Worth knowing because it needs nothing installed, which matters exactly when you're on a machine you've just started troubleshooting. tmux is better when you want to come back to a live terminal — but for fire-and-watch-the-log jobs, it's a dependency you don't need.
