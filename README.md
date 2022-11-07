# action-swift-cli-build

Build swift cli tool on macOS or ubuntu, to run it or attach it to release

## Build and use

```yaml
jobs:
  build:
    name: Swift on ${{ matrix.os }}
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest]
    steps:
    - uses: phimage/action-swift-cli-build@v1
      with:
        swift-version: "5.7"
        bin-name: "my-binary-name"
    - name: Launch my compile cli tools
      run: |
        $BINARY --help
```

## Release

To upload to a release when publishing it, use `upload-to-release` and pass the `repo-token`.

```yaml
name: release

on: 
  release:
    types: [published]

jobs:
  build:
    name: Swift on ${{ matrix.os }}
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest]
    steps:
    - uses: phimage/action-swift-cli-build@v1
      with:
        swift-version: "5.7"
        bin-name: "my-binary-name"
        upload-to-release: true
        repo-token: ${{ secrets.GITHUB_TOKEN }}
```

### Propose an install bash script

A user then could launch a simple command to install your binary from release assets

```bash
sudo curl -sL https://<orga>.github.io/<project>/install.sh | bash
```

To do so, first active the [github pages](https://docs.github.com/en/pages/getting-started-with-github-pages/about-github-pages).

Then create an "install.sh" script in your project and do not forget to edit `binary` and `repository` variable to match your project needs.

```bash
#!/usr/bin/env bash

binary_name="yourbinary"
repository="orga/project"

TMP="${TMPDIR}"
if [ "x$TMP" = "x" ]; then
  TMP="/tmp/"
fi
TMP="${TMP}cli.$$"
rm -rf "$TMP" || true
mkdir "$TMP"
if [ $? -ne 0 ]; then
  echo "failed to mkdir $TMP" >&2
  exit 1
fi

cd $TMP

if [[ "$OSTYPE" == "linux-gnu" ]]; then
  . /etc/lsb-release
  if [ "$DISTRIB_ID" != "Ubuntu" ]; then
    echo "Only ubuntu supported"
    exit 1
  fi
  archiveName=$binary_name-x86_64-static-ubuntu-$DISTRIB_CODENAME.zip
elif [[ "$OSTYPE" == "darwin"* ]]; then  # Mac OSX
  archiveName=$binary_name.zip
else
  echo "Unknown os type $OSTYPE, macOS or ubuntu"
  exit 1
fi

archive=$TMP/$archiveName
curl -sL https://github.com/$repository/releases/latest/download/$archiveName -o $archive

if [[ "$OSTYPE" == "darwin"* ]]; then  # Mac OSX
  unzip -q $archive -d $TMP/
else
  unzip -q $archive -d $TMP/
fi

binary=$TMP/$binary_name 

dst="/usr/local/bin"
echo "Install into $dst/binary_name"
sudo rm -f $dst/$binary_name
sudo cp $binary $dst/

rm -rf "$TMP"
```
