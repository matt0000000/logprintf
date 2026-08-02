+++
title = "read the full error message"
date = "2026-03-31"
description = "validation errors often tell you exactly what is wrong (shocking i know)"
tags = [
"debugging",
"configuration"
]
+++

I sometimes focus on the feature that is not working and overlook the actual error message.

In one case, the application clearly said:

```text
Expected string, got array
```

That was the problem.

Before searching for complicated fixes, read the complete error and compare the received value with the expected type.

