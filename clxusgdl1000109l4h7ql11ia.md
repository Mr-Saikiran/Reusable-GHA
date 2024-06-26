---
title: "Cache Docker, Python, Java(Maven), Node(npm), Go Modules in Github Actions"
seoTitle: "Github Actions Caching"
seoDescription: "Improve GitHub Actions with efficient caching techniques for Docker, Python, Java, Node, and Go to speed up workflows"
datePublished: Thu May 11 2023 18:30:00 GMT+0000 (Coordinated Universal Time)
cuid: clxusgdl1000109l4h7ql11ia
slug: cache-docker-python-javamaven-nodenpm-go-modules-in-github-actions
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1719430177660/a66047c7-a495-4b2f-bfef-54aa42a437c8.jpeg
ogImage: https://cdn.hashnode.com/res/hashnode/image/upload/v1719342982956/e51bb9bc-797f-43aa-be8b-4981218ef623.jpeg
tags: workflow, caching, cache, ci-cd, github-actions-1, caching-strategies

---

"Caching helps workflows run faster". The cache is created during the first run and is used for later runs, saving time and keeping the workflow efficient by speeding up the workflow runs.

## Cache Maven

Maven usually takes a longer time to build the dependencies because it needs to download all the required libraries and plugins from remote repositories. This process can be significantly sped up by caching the Maven repository. To achieve this, you can set the path to the local Maven repository and use the `key` and `restore-keys` arguments in your caching configuration. By doing so, the dependencies are stored locally after the first run and can be reused in subsequent runs, eliminating the need to download them again. This approach not only saves time but also reduces the load on remote repositories, making your build process more efficient and reliable.

```yaml
- name: Cache Maven
  uses: actions/cache@v4
  with:
    path: ~/.m2/repository
    key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
    restore-keys: |
      ${{ runner.os }}-m2-
```

## Cache NodeJS

In Unix-based Operating Systems, NodeJS npm cache files are stored under `~/.npm`. This directory contains all the cached files that npm uses to speed up subsequent installations of packages. By caching these files, you can significantly reduce the time it takes to install dependencies in future runs, as npm will reuse the cached files instead of downloading them again. This is particularly useful in continuous integration and deployment pipelines, where the same dependencies are often installed repeatedly. To cache the npm files, you can use the following configuration in your workflow:

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-
      ${{ runner.os }}-
```

Alternatively, this can be used directly with the `setup-node` action.

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: '20.x'
    cache: 'npm'
    cache-dependency-path: '**/package-lock.json'
```

## Cache Python

Python dependencies can be cached to speed up workflow runs significantly. For `pipenv`, the dependencies are typically stored under `~/.local/share/virtualenvs`. This directory contains the virtual environments created by `pipenv`, which include all the necessary packages and dependencies for your Python projects. By caching this directory, you can avoid the time-consuming process of reinstalling these dependencies during each workflow run.

**for PIPENV**

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.local/share/virtualenvs
    key: ${{ runner.os }}-python-${{ steps.setup-python.outputs.python-version }}-pipenv-${{ hashFiles('Pipfile.lock') }}
```

**for PIP**  

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.cache/pip
    key: ${{ runner.os }}-pip-${{ hashFiles('**/requirements.txt') }}
    restore-keys: |
      ${{ runner.os }}-pip-
```

## Cache Docker

Significantly speed up your Docker builds by leveraging GitHub's infrastructure to store and retrieve cache layers. Docker provides multiple caching options, including Registry Cache, Inline Cache, and GitHub Cache. Each method has its own advantages, but I recommend using GitHub Cache with the `type=gha` as it is integrated seamlessly with GitHub Actions, making it easier to set up and manage within your CI/CD workflows.

```yaml
    - name: Build and push
      uses: docker/build-push-action@v6
      with:
        # context: docker-springboot
        context: .
        push: true
        tags: |
          ${{ inputs.acr}}/${{ github.event.repository.name }}:latest
        # cache-from: type=registry,ref=${{ inputs.acr-url }}/${{ env.containerName }}:buildcache
        # cache-to: type=registry,ref=${{ inputs.acr-url }}/${{ env.containerName }}:buildcache,mode=max
        cache-from: type=gha
        cache-to: type=gha,mode=max
```

## Cache Go

Like other builds, Go builds can benefit significantly from caching. By storing and reusing dependencies and build artifacts, we can drastically reduce build times and improve efficiency. GitHub Actions provides a convenient way to cache Go modules and build outputs, ensuring that subsequent builds are faster and more reliable.

```yaml
- uses: actions/cache@v4
  with:
    path: |
      ~/.cache/go-build
      ~/go/pkg/mod
    key: ${{ runner.os }}-go-${{ hashFiles('**/go.sum') }}
    restore-keys: |
      ${{ runner.os }}-go-
```

Similarly, we can use the caching directly using `setup-go` action

```yaml
  - uses: actions/setup-go@v5
    with:
      go-version: '1.17'
      check-latest: true
      cache-dependency-path: "**/*.sum"
```

You can find more information and detailed instructions on how to use the caching feature by visiting the [Cache action](https://github.com/marketplace/actions/cache) page.

By following the above examples, you can ensure that your builds are not only faster but also more efficient and reliable. Whether you are working with Go, Node.js, Python, or any other language, the Cache action can help you to implement effective caching in your GitHub Actions workflows.