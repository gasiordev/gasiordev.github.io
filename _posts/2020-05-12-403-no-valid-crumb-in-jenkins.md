---
layout: post
title: "403 No valid crumb... when calling Jenkins"
author: "Mikolaj Gasior"
---

There are two steps to trigger a jenkins build: 
1. `GET https://yourjenkins/crumbIssuer/api/xml?xpath=concat((//crumbRequestField,":",//crumb)` with basic auth;
2. `POST https://yourjenkins/job/myjob1/build/` with basic auth and `content-type` header and crumb header received in first request (eg. `Jenkins-Crumb: aabbccdd`).

However, if you updated Jenkins to 2.222 you may encounter that it stopped
to work and `build` endpoint returns
`403 No valid crumb was included in the request`.

There are few ways to fix this and the easiest one is to use a plugin called
*Strict Crumb Issuer*.

After installing it, it has to be selected in *Manage Jenkins > Global Settings > CSRF Protection*.
In the same section, there's a button labelled *Advanced* which shows plugin's
configuration. Uncheck the *Check the session ID* tickbox.

If you need more details on how to trigger a build look at the source of
[github-webhookd](https://github.com/gasiordev/github-webhookd/blob/master/jenkinsapi.go).

