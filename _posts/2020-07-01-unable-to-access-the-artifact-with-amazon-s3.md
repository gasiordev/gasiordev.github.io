---
layout: post
title: "Unable to access the artifact.. when executing Code Pipeline to build an image"
author: "Mikolaj Gasior"
---

When you build Code Pipeline and one of the steps is a Code Build block that
builds a docker image that is pushed to a docker registry (eg. ECR), you don't
really need to build the artifact that is populated further. It's that image
that's now in the registry and Code Deploy step will just pull it.

However, you might encounter the following error:

*Unable to access the artifact with Amazon S3 object key 
'something/build_outp/XXXXXXX' located in the Amazon S3 artifact bucket 
'xxxxxx'. The provided role does not have sufficent permissions.*

It might be a bit confusing (assuming that permissions are all right).

What happens to be the missing bit is the [Image Definitions](https://docs.aws.amazon.com/codepipeline/latest/userguide/file-reference.html)
file.

In your `buildspec.yml` [file](https://docs.aws.amazon.com/codebuild/latest/userguide/build-spec-ref.html),
ensure you have the following lines:

```
  post_build:
    commands:
      ...
      - printf '[{"name":"%s","imageUri":"%s"}]' $IMAGE_REPO_NAME $AWS_ACCOUNT_ID.dkr.ecr.$AWS_DEFAULT_REGION.amazonaws.com/$IMAGE_REPO_NAME:$IMAGE_TAG > imagedefinitions.json
  artifacts:
    files: imagedefinitions.json
```

Now, the artifact will be used to tell Code Deploy what docker image needs to 
be released.

