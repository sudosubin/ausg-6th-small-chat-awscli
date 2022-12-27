<!-- omit in toc -->
# ausg-6th-small-chat-awscli

Here introduces some useful cli snippets & scenarios when using awscli. [Presentation (Korean)](https://docs.google.com/presentation/d/1xZAhiTv4wdW5MnyBRQ9dEW0WKmhUzYWv9WuSSwJdP5s/edit?usp=sharing) presented on 2022-11-14.

<!-- omit in toc -->
## Table of Contents

- [First awscli output](#first-awscli-output)
- [Using output option](#using-output-option)
- [Extract fields (jq)](#extract-fields-jq)
- [Extract fields (query)](#extract-fields-query)
- [More usecases](#more-usecases)
- [Referenecs](#referenecs)

## First awscli output

The example below is a bit poor to use. JSON is great, but a bit tricky to deal with in the world of shell scripting.

```sh
$ aws ecr list-images --repository-name curl
{
    "imageIds": [
        {
            "imageDigest": "sha256:71cf92d...",
            "imageTag": "v1.5"
        },
        {
            "imageDigest": "sha256:5ddbcd0",
            "imageTag": "infinity"
        }
    ]
}
```

## Using output option

There is an option to change output in plain text or yaml, but it is still a bit difficult.

```sh
$ aws ecr list-images --repository-name curl --output text
IMAGEIDS        sha256:71cf92d... v1.5
IMAGEIDS        sha256:5ddbcd0... infinity

$ aws ecr list-images --repository-name curl --output yaml
imageIds:
- imageDigest: sha256:71cf92d...
  imageTag: v1.5
- imageDigest: sha256:5ddbcd0...
  imageTag: infinity
```

## Extract fields (jq)

Using jq? However, CI/CD may not have jq. You may not want to install jq separately. Is there any simpler way?

```sh
$ aws ecr list-images --repository-name curl | jq '.imageIds[].imageTag'
"v1.5"
"infinity"
```

## Extract fields (query)

That's the awscli query option.

```sh
$ aws ecr list-images --repository-name curl --query 'imageIds[].imageTag'
[
    "v1.5",
    "infinity"
]

$ aws ecr list-images --repository-name curl --query 'imageIds[].[imageTag]' --output text
v1.5
infinity
```

## More usecases

Great.

```sh
$ for arn in $(aws ecr describe-repositories \
        --query 'repositories[].[repositoryArn]' \
        --output text); do
    echo "[info] processing $arn"
    aws ecr tag-resource --resource-arn "$arn" --tags "user:Company=global"
done

[info] processing arn:aws:ecr:ap-northeast-2:***:repository/curl
[info] processing arn:aws:ecr:ap-northeast-2:***:repository/openjdk
...
```

## Referenecs

Below document contains all about awscli output filtering ways. There also awscli server-side filtering is available. (when the api supports)
- https://docs.aws.amazon.com/cli/latest/userguide/cli-usage-filter.html
