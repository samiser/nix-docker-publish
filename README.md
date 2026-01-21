# Nix Docker Image Publisher

A GitHub Action for building and publishing Docker images from Nix flake
outputs to the GitHub Container Registry (ghcr.io).

## Features

- Simple integration with Nix flakes
- Automatic tagging with commit SHA
- Optional `latest` tag for main/master branches

## Usage

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

## Inputs

| Input          | Description                                | Required | Default                    |
| -------------- | ------------------------------------------ | -------- | -------------------------- |
| `image-name`   | Image name (defaults to repository name)   | No       | `${{ github.repository }}` |
| `flake-output` | Nix flake output path for the Docker image | No       | `.#dockerImage`            |
| `source-tag`   | Nix installer version tag                  | No       | `v0.31.0`                  |
| `github-token` | GitHub token for registry authentication   | **Yes**  | -                          |
| `tag-latest`   | Whether to tag as `latest` on main/master  | No       | `true`                     |

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

1. **Always**: `ghcr.io/{image-name}:{commit-sha}`
2. **On main/master** (if `tag-latest: true`): `ghcr.io/{image-name}:latest`

## Example: Multiple Flake Outputs

```yaml
jobs:
  publish-api:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4
      - uses: samiser/nix-docker-publish@v1.0.0
        with:
          flake-output: ".#apiImage"
          image-name: ${{ github.repository }}/api
          github-token: ${{ secrets.GITHUB_TOKEN }}

  publish-web:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - uses: actions/checkout@v4
      - uses: samiser/nix-docker-publish@v1.0.0
        with:
          flake-output: ".#webImage"
          image-name: ${{ github.repository }}/web
          github-token: ${{ secrets.GITHUB_TOKEN }}
```
