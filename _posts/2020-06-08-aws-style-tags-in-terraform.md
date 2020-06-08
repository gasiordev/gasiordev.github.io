---
layout: post
title: "AWS style tags in Terraform"
author: "Mikolaj Gasior"
---

With version 0.12 important syntax changes were introduced to Terraform and the following code containing AWS style tags is no longer valid.

```
tags {
  "gsr:env" = "prod"
  "gsr:name" = "my-instance"
  "gsr:tag" = "value"
  ...
}
```

Tag keys cannot be quoted and have colon. A quick hack to it is to use `merge` function like below:

```
tags = merge(
  {
    "gsr:env" = "prod
  },
  {
    "gsr:name" = "my-instance",
    "gsr:tag" = "value",
    ...
  }
)
```

Hope that helps!

