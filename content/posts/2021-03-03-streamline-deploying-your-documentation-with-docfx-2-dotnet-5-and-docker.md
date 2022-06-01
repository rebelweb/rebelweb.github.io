---
title: Streamline Deploying your documentation with DocFx 2, DotNet 5, and Docker
date: 2021-03-03 22:08 -500
draft: false
---

At the time of writing this DocFx is not playing well DotNet (.NET) 5
Projects. The error specifically is reading .csproj and .sln files to
get the API documentation, or your 3 slash code comments. This presented
me with a slight problem, and here was my solution.

I was needing to publish my documentation for a work project since we 
just upgraded to .NET 5. So I first was working on running Docfx with
Mono inside a docker container it took me a few minutes to get that
running after getting that to successful work I came across the bug
between .NET 5 & Docfx, noted here: Github. So you just have to build
your code, let docfx read those .dll files, then finally deploy to a 
web server for view.

To achieve this I built a 3 stage docker file to get the documentation
built you can see an example of the Dockerfile below.

```Dockerfile
# build .dll files for docfx to inspect
FROM mcr.microsoft.com/dotnet/sdk:5.0.102-alpine3.12-amd64 AS code-env

WORKDIR /app

COPY . .

RUN dotnet publish -o ./.build src/FileShare.API

# build docs with docfx using mono
FROM mono as docs-env

RUN apt update && apt install -y unzip wget

WORKDIR /tools

RUN wget https://github.com/dotnet/docfx/releases/download/v2.56.7/docfx.zip

RUN unzip docfx.zip -d ./docfx

WORKDIR /build

COPY --from=code-env /app/.build .build

COPY . .

RUN mono /tools/docfx/docfx.exe build docfx_project/docfx.json

# deploy using nginx for static file hosting
FROM nginx:latest

WORKDIR /var/www/html

COPY --from=docs-env /build/docfx_project/_site .

WORKDIR /etc/nginx/conf.d

COPY ./docfx_project/default.conf default.conf
```

## Why I use Docker for documentation deployment

I have used docker deploying services for some time now. I like that
there is no configuring the service on a server, or a developer on 
another team, can pull it and run it locally, so they have access 
offline. It also integrates well in my build pipeline.