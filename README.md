# 33090

Reproduction for [Renovate issue 33090](https://github.com/renovatebot/renovate/discussions/33090).

## Current behavior

When setting a registry for datasources gitlab-releases
```json
"packageRules": [
  {
    "description": [
      "Configure registry URLs"
    ],
    "matchDatasources": [
      "gitlab-releases",
      "gitlab-tags",
      "gitlab-packages"
    ],
    "registryUrls": [
      "https://self-hosted-gitlab.com/"
    ]
  }
]
```

it is used even when the renovate configuration sets different registry, i.e.
by this comment in the Dockerfile.

```Dockerfile
FROM scratch
# renovate: datasource=gitlab-releases depName=components/opentofu versioning=semver registryUrl=https://gitlab.com
ENV GITLAB_TOFU_VERSION="0.21.0"
```

According to the logs, the registryUrl is found to be https://gitlab.com/ for the above
dependency

```logs
  "config": {
    "regex": [
      {
        "deps": [
          {
            "depName": "components/opentofu",
            "currentValue": "0.21.0",
            "datasource": "gitlab-releases",
            "versioning": "semver",
            "registryUrls": [
              "https://gitlab.com/"
            ],
            "replaceString": "# renovate: datasource=gitlab-releases depName=components/opentofu versioning=semver registryUrl=https://gitlab.com\nENV GITLAB_TOFU_VERSION=\"0.21.0\"\n",
            "updates": [],
            "packageName": "components/opentofu",
            "warnings": [
              {
                "topic": "components/opentofu",
                "message": "Failed to look up gitlab-releases package components/opentofu"
              }
            ]
          }
        ],
        "matchStrings": [
          "# renovate: datasource=(?<datasource>[a-z-.]+?) depName=(?<depName>[^\\s]+?)(?: (lookupName|packageName)=(?<packageName>[^\\s]+?))?(?: versioning=(?<versioning>[^\\s]+?))?(?: extractVersion=(?<extractVersion>[^\\s]+?))?(?: registryUrl=(?<registryUrl>[^\\s]+?))?\\s(?:ENV|ARG)\\s+[A-Za-z0-9_]+?_VERSION[ =][\"']?(?<currentValue>.+?)[\"']?\\s"
        ],
        "packageFile": "Dockerfile"
      }
    ]
  }
}
```

But the request renovate performs goes against https://self-hosted-gitlab.com

```logs
DEBUG: Gitlab API error
{
  "err": {
    "name": "RequestError",
    "code": "ENOTFOUND",
    "timings": {
      "start": 1734093273803,
      "socket": 1734093273804,
      "lookup": 1734093273808,
      "error": 1734093273809,
      "phases": {
        "wait": 1,
        "dns": 4,
        "total": 6
      }
    },
    "message": "getaddrinfo ENOTFOUND self-hosted-gitlab.com",
    "stack": "RequestError: getaddrinfo ENOTFOUND self-hosted-gitlab.com\n    at ClientRequest.<anonymous> (/usr/local/renovate/node_modules/.pnpm/got@11.8.6/node_modules/got/dist/source/core/index.js:970:111)\n    at Object.onceWrapper (node:events:633:26)\n    at ClientRequest.emit (node:events:530:35)\n    at ClientRequest.emit (node:domain:489:12)\n    at ClientRequest.origin.emit (/usr/local/renovate/node_modules/.pnpm/@szmarczak+http-timer@4.0.6/node_modules/@szmarczak/http-timer/dist/source/index.js:43:20)\n    at emitErrorEvent (node:_http_client:103:11)\n    at TLSSocket.socketErrorListener (node:_http_client:506:5)\n    at TLSSocket.emit (node:events:518:28)\n    at TLSSocket.emit (node:domain:489:12)\n    at emitErrorNT (node:internal/streams/destroy:170:8)\n    at emitErrorCloseNT (node:internal/streams/destroy:129:3)\n    at processTicksAndRejections (node:internal/process/task_queues:90:21)\n    at GetAddrInfoReqWrap.onlookupall [as oncomplete] (node:dns:120:26)",
    "options": {
      "headers": {
        "user-agent": "RenovateBot/39.58.1 (https://github.com/renovatebot/renovate)",
        "accept": "application/json",
        "accept-encoding": "gzip, deflate, br"
      },
      "url": "https://self-hosted-gitlab.com/api/v4/projects/components%2Fopentofu/releases",
      "hostType": "gitlab-releases",
      "username": "",
      "password": "",
      "method": "GET",
      "http2": false
    }
  }
}
```

## Expected behavior

When checking the mentioned gitlab-releases dependency, requests are performed
against the stated registryUrl, namely https://gitlab.com.

## Link to the Renovate issue or Discussion

Link to the Renovate Discussion:
[https://github.com/renovatebot/renovate/discussions/33090](git@github.com:uluzox/renovate-minimal-reproduction-gitlab-registry-url.git)
