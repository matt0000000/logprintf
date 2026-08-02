+++
title = "Two SMART logs, two different stories"
date = "2026-04-04"
description = "A clean error log on a drive that kept failing self-tests"
tags = [
"smartctl",
"hardware",
"selfhosted"
]
+++
A drive in my pool was failing extended self-tests while the SMART error log stayed completely clean. It reads like a contradiction and I nearly talked myself into trusting the clean log.

They track different things. The error log records command-level ATA errors the firmware explicitly reports — unrecoverable reads, write faults. The self-test log records whether the test procedure completed. A drive can abort a test early on some internal condition without ever generating a loggable error, which is exactly what "unknown failure" at 90% remaining means: it gave up almost immediately and had nothing to file.

The self-test result is the one to believe. Combined with reallocated sectors it was enough to start the migration, and no, firmware updates don't repair physical sectors.
