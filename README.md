# Cache Anything New in Container | Action

This is a fork of [airvzxf/cache-anything-new-action](https://github.com/airvzxf/cache-anything-new-action) that works in Docker containers.

`sudo` does not work in Docker containers, so instead run the container as root (with `options: --user 0`).

This GitHub action scans the container to check if anything new was added after you run a script, then caches all new files.

## Usage example

Create your script as `.github/workflows/install.sh`

```bash
#!/bin/bash -e

sudo apt update
sudo apt install -y pandoc
echo "Hello World!" > hello.txt
```

Add the action in `.github/workflows/your_action.yml`.

```yaml
name: CI - Testing Cache Anything New | Action
on:
  workflow_dispatch:
jobs:
  testing_actions:
    name: Testing Actions | Job
    runs-on: 'ubuntu-latest'
    container: 
      image: 'ubuntu:16.04'
      options: --user 0
    steps:
      - uses: actions/checkout@v2

      # id: will be used in the cache anything action.
      # path: should always start with runner.temp
      - name: Generate cache
        uses: actions/cache@v2
        id: cache-id
        with:
          path: ${{ runner.temp }}/cache-directory-example
          key: ${{ runner.os }}-cache-hello-world-key-v1.0

      # is_cached: use the previous cache action id.
      # cache: should be the same as the cache 'path'.
      - name: Cache anything
        uses: plu5/cache-anything-new-in-container-action@main
        with:
          script: 'install.sh'
          is_cached: ${{ steps.cache-id.outputs.cache-hit }}
          cache: ${{ runner.temp }}/cache-directory-example
          snapshot: '/'
          exclude: '/boot /data /dev /mnt /proc /run /sys'

      # Optional
      - name: Use the cache
        run: |
          find /home/ -iname hello.txt 2> /dev/null || true
          whereis pandoc || true
```

The `uses: actions/cache` is required because it needs this external action to store the cache. This action only finds the new created files and stores them in a temporary directory which is cached by `actions/cache`.

## Settings

Option | Description | Required | Default | Example
---    | ---         | ---      | ---     | ---
script | Your script.<br> This script needs to be the same directory as your action: `.github/workflows/`. | Yes | N/A | `install.sh`
is_cached | Boolean value to check if the cache was created.<br> It should be the same as the `actions/cache` `id`. | Yes | N/A | `${{ steps.cache-id.outputs.cache-hit }}`
cache | The directory where the new detected files will be stored.<br> It should be the same as the `actions/cache` `path`. | Yes | N/A | `${{ runner.temp }}/cache-directory-example`
snapshot | The path of the scanner entry point.<br> It can scan from the main path `/` or some specific directory. | No | / | /home/runner
exclude | Exclude directories or files that don’t need to be cached.<br> Separate with spaces. | No | /boot /data /dev /mnt /proc /run /sys | /root /home/github /actions /home/runner/.bashrc
