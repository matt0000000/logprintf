+++
title = "rsync couldn't delete what it copied"
date = "2026-06-26"
description = "Migrating 2TB off a failing drive in a MergerFS pool"
tags = [
"mergerfs",
"linux",
"selfhosted"
]
+++
One of the drives in my MergerFS pool started showing pending sectors, so I mounted it read-only and rsynced everything off with `--remove-source-files`.

Two terabytes later, rsync finished the copy and then threw errors on every single delete. Obvious in hindsight: read-only was the whole point of the mount, and `--remove-source-files` needs write access to unlink the originals. The data was safely on the other side, it just couldn't clean up after itself.

Remounted with `mount -o remount,rw` and ran the same command again. rsync skips anything already transferred, so the second pass was just the deletions and took minutes. Worth knowing before you panic at a screen full of errors.
