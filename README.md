# Nix Docker Image Publisher

A GitHub Action for building and publishing Docker images from Nix flake
outputs.

## Features

- Simple integration with Nix flakes
- Automatic tagging with commit SHA
- Optional `latest` tag for main/master branches
- Customizable registry and image names

## Usage

### Basic Example

```yaml
name: Publish Docker Image

on:
  push:
    branches: [main]

jobs:
  publish:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4

      - name: Publish Docker image
        uses: samiser/nix-docker-publish@v1.0.0
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
```

### Advanced Example

```yaml
- name: Publish Docker image
  uses: samiser/nix-docker-publish@v1.0.0
  with:
    registry: ghcr.io
    image-name: my-org/my-app
    flake-output: ".#docker"
    github-token: ${{ secrets.GITHUB_TOKEN }}
    tag-latest: true
```

## Inputs

| Input               | Description                                | Required | Default                    |
| ------------------- | ------------------------------------------ | -------- | -------------------------- |
| `registry`          | Container registry URL                     | No       | `ghcr.io`                  |
| `image-name`        | Image name (defaults to repository name)   | No       | `${{ github.repository }}` |
| `flake-output`      | Nix flake output path for the Docker image | No       | `.#dockerImage`            |
| `nix-installer-tag` | Nix installer version tag                  | No       | `v0.31.0`                  |
| `github-token`      | GitHub token for registry authentication   | **Yes**  | -                          |
| `tag-latest`        | Whether to tag as `latest` on main/master  | No       | `true`                     |

## Outputs

| Output   | Description                   |
| -------- | ----------------------------- |
| `image`  | Full image name with registry |
| `digest` | Image digest                  |

## Requirements

Your Nix flake must have a `dockerImage` output (or whatever you specify in
`flake-output`). Example:

```nix
{
  outputs = { self, nixpkgs, ... }: {
    dockerImage = nixpkgs.legacyPackages.x86_64-linux.dockerTools.buildImage {
      name = "my-app";
      tag = "latest";
      config = {
        Cmd = [ "${self.packages.x86_64-linux.default}/bin/my-app" ];
      };
    };
  };
}
```

## Tagging Strategy

The action applies the following tags:

1. **Always**: `{registry}/{image-name}:{commit-sha}`
2. **On main/master** (if `tag-latest: true`): `{registry}/{image-name}:latest`

## Examples

### Using with custom registry

```yaml
- uses: samiser/nix-docker-publish@v1.0.0
  with:
    registry: docker.io
    image-name: myuser/myapp
    github-token: ${{ secrets.DOCKERHUB_TOKEN }}
```

### Multiple flake outputs

```yaml
jobs:
  publish-api:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: samiser/nix-docker-publish@v1.0.0
        with:
          flake-output: ".#apiImage"
          image-name: ${{ github.repository }}/api
          github-token: ${{ secrets.GITHUB_TOKEN }}

  publish-web:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: samiser/nix-docker-publish@v1.0.0
        with:
          flake-output: ".#webImage"
          image-name: ${{ github.repository }}/web
          github-token: ${{ secrets.GITHUB_TOKEN }}
```
