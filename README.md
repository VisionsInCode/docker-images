﻿# Repository of Sitecore Docker images

[//]: # "start: stats"

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg?style=flat-square)](https://opensource.org/licenses/MIT) ![Repositories](https://img.shields.io/badge/Repositories-58-blue.svg?style=flat-square) ![Tags](https://img.shields.io/badge/Tags-136-blue.svg?style=flat-square) ![Deprecated](https://img.shields.io/badge/Deprecated-0-lightgrey.svg?style=flat-square) ![Dockerfiles](https://img.shields.io/badge/Dockerfiles-52-blue.svg?style=flat-square) ![Default version](https://img.shields.io/badge/Default%20version-9.2.0%20on%20ltsc2019/1809-blue?style=flat-square)

[//]: # "end: stats"

Build your own Docker images for the most recent versions of Sitecore. See [IMAGES.md](IMAGES.md) for all images currently available. You can also use this repository (preferably from a fork) from you own build server and have it build and push images to your own private Docker registry.

Jump to the [How to use](#how-to-use) section to get started.

## IMPORTANT NOTES ABOUT THIS REPOSITORY

- This repository was created to help consolidate efforts around Sitecore and Docker.
- **The code and examples found in this repository are created and maintained by the Community,  unsupported by Sitecore.**
- Official statement from Sitecore on *running* in containers, see [https://kb.sitecore.net/articles/161310](https://kb.sitecore.net/articles/161310).

### Change Log

Please see [CHANGELOG.md](CHANGELOG.md).

### List of all images

Please see [IMAGES.md](IMAGES.md).

### Tagging and Windows versions

This repository can build multiple Windows versions. Read more about [Windows Container Version Compatibility](https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/version-compatibility).

Here is the convention used for Sitecore image tags:

```text
 [REGISTRY/]sitecore-<TOPOLOGY>[-VARIANT]-<ROLE>:<SITECORE_VERSION>-<OS_VERSION>
```

Example:

```text
 registry.example.com/sitecore-xm-cm:9.2.0-windowsservercore-1903
 \__________________/ \____________/ \___/ \____________________/
           |                 |         |             |
   registry/org/user    repository  sc version   os version
```

## How to use

### Quick start

```PowerShell
.\Build.ps1 -SitecoreUsername "YOUR dev.sitecore.net USERNAME" -SitecorePassword "YOUR dev.sitecore.net PASSWORD"
```

This will:

1. Download any missing packages into `.\packages`, if you have another location with files already present you can call `Build.ps1` with the parameter `-InstallSourcePath`.
1. Build all images of latest Sitecore version on latest LTSC (Long Term Support Channel) Windows version.

> Images will always be saved locally but not pushed to any remote registries by default. See [Setting up automated builds](#setting-up-automated-builds) for details on how to do this.

When completed:

1. Place your Sitecore license file at `C:\license\license.xml`, or override location using the environment variable `LICENSE_PATH` like so: `$env:LICENSE_PATH="D:\my\sitecore\licenses"`
1. Switch directory to .\windows\tests\9.x.x\ Then run any of the docker-compose files, for example an XM with: `docker-compose --file .\docker-compose.xm.yml up`

### Setting up automated builds

### Prerequisites

- A **private** Docker repository. Any will do, but the easiest is to use a [Azure Container Registry](https://azure.microsoft.com/en-us/services/container-registry/).
- A file location that your build agents can reach to store downloads from [https://dev.sitecore.net/](https://dev.sitecore.net/).

#### Windows

1. Latest Windows 10 or Windows Server 2019 with Hyper-V and Containers features installed.
1. Latest stable Docker engine and cli.

#### Linux (optional)

1. [PowerShell Core](https://github.com/powershell/powershell) so you can use the PowerShell module.
1. Latest stable Docker engine and cli.

### Configure your build server

1. Trigger a build on changes to `master` - to get new versions.
1. Trigger once a week - to get base images updated when Microsoft releases patched images.

Example:

```PowerShell

# required, change if you need to build images in other folders such as ".\linux" or ".\legacy"
$imagesPath = (Join-Path $PSScriptRoot "\windows")

# optional, default value is ".\packages". Can be on local machine or a file share.
$installSourcePath = (Join-Path $PSScriptRoot "\packages")

# optional, on Docker Hub it's your username or organization, else it's the hostname of your
# own registry. This parameter is optional but you will not be able to push images to a
# remote registry without.
#
# PLEASE NOTE: DO NOT SPECIFY A PUBLIC REGISTRY!
#
$registry = "YOUR REGISTRY NAME" `

# optional, default value is the latest Sitecore version on latest LTSC version
# of Windows. Set to for example "*" for build everything or "*:9.1.1*1903", "*:9.2.0*1903" to
# only build 9.1.1 and 9.2.0 on Windows 1903.
$tags = "*"

# required
$sitecoreUsername = "YOUR dev.sitecore.net USERNAME"

# required
$sitecorePassword = "YOUR dev.sitecore.net PASSWORD"

# import builder module
Import-Module (Join-Path $PSScriptRoot "\modules\SitecoreImageBuilder") -Force

# restore packages needed for the build, only files missing in $installSourcePath will be downloaded
SitecoreImageBuilder\Invoke-PackageRestore `
    -Path $imagesPath `
    -Destination $installSourcePath `
    -Tags $tags `
    -SitecoreUsername $sitecoreUsername `
    -SitecorePassword $sitecorePassword

# build and push images
SitecoreImageBuilder\Invoke-Build `
    -Path $imagesPath `
    -InstallSourcePath $installSourcePath `
    -Registry $registry `
    -Tags $tags
```
