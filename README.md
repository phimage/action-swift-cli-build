# action-swift-cli-build-

Build swift cli tool on macOS or ubuntu

## Build

```yaml
jobs:
  build:
    name: Swift ${{ matrix.swift }} on ${{ matrix.os }}
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest]
        swift: ["5.7"]
    steps:
    - uses: phimage/action-swift-cli-build@v1
      id: binary
      with:
        swift-version: ${{ matrix.swift }}
        bin-name: "my-binary-name"
    - name: Launch my compile cli tools
      run: |
        ${{ steps.binary.outputs.binary }} --help
```

## Release

To upload to release when published, use `upload-to-release` and pass `repo-token`.

```yaml
name: release

on: 
  release:
    types: [published]

jobs:
  build:
    name: Swift ${{ matrix.swift }} on ${{ matrix.os }}
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest]
        swift: ["5.7"]
    steps:
    - uses: phimage/action-swift-cli-build@v1
      with:
        swift-version: ${{ matrix.swift }}
        bin-name: "my-binary-name"
        upload-to-release: true
        repo-token: ${{ secrets.GITHUB_TOKEN }}
```
