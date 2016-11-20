Known issues & workarounds
==========================

- [Preview 2 tooling known issues](#preview-2-tooling-known-issues)
    - [OpenSSL dependency on OS X](#openssl-dependency-on-os-x)
    - [`brew` refusing to link `openssl`](#brew-refusing-to-link-openssl)
    - [Running .NET Core CLI on Nano Server](#running-net-core-cli-on-nano-server)
    - [Users of zsh (z shell) don't get `dotnet` on the path after install](#users-of-zsh-z-shell-dont-get-dotnet-on-the-path-after-install)
    - [`app.config` file needs to be checked out before publishing](#appconfig-file-needs-to-be-checked-out-before-publishing)
    - [`dotnet` commands in the root of the file system fails](#dotnet-commands-in-the-root-of-the-file-system-fails)
    - [On dev builds of the tools, restoring default project from dotnet new fails](#on-dev-builds-of-the-tools-restoring-default-project-from-dotnet-new-fails)
    - [Running `dotnet` on Debian distributions causes a segmentation fault](#running-dotnet-on-debian-distributions-causes-a-segmentation-fault)
    - [Uninstalling/reinstalling the PKG on OS X](#uninstallingreinstalling-the-pkg-on-os-x)
- [Preview 3 tooling known issues](#preview-3-tooling-known-issues)
    - [Installing VS 2017 RC or dotnet CLI preview3 prevents Visual Studio 2015 .NET Core tooling from working](#installing-vs-2017-rc-or-dotnet-cli-preview3-prevents-visual-studio-2015-net-core-tooling-from-working)
    - [Restore required before using the .NET Core tooling Preview 3](#restore-required-before-using-the-net-core-tooling-preview-3)
    - [`dotnet test` has changed from Preview 2 `dotnet test`](#dotnet-test-has-changed-from-preview-2-dotnet-test)
- [What is this document about?](#what-is-this-document-about)
- [What is a "known issue"?](#what-is-a-known-issue)

# Preview 2 tooling known issues

## OpenSSL dependency on OS X
OS X "El Capitan" (10.11) comes with 0.9.8 version of OpenSSL. .NET Core depends on versions >= 1.0.1 of OpenSSL. You can update the version by using [Homebrew](https://brew.sh), [MacPorts](https://www.macports.org/) or manually. The important bit is that you need to have the required OpenSSL version on the path when you work with .NET Core. 

With Homebrew, you can run the following commands to get this done: 

```console
brew update
brew install openssl
```

Homebrew may also show the following warning:

> Apple has deprecated use of OpenSSL in favor of its own TLS and crypto libraries

This warning is meant for the software that uses OpenSSL (in this case, .NET Core) and not for the end-user that is installing said software. Homebrew installation doesn't touch either the existing Apple crypto libraries or existing OpenSSL 0.9.8 version, so there is no impact on any software that uses either one of those crypto solutions and is already installed.

MacPorts doesn't have the concept of linking, so it is reccomended that you uninstall 0.9.8 version of OpenSSL using the following command:

```console
sudo port upgrade openssl
sudo port -f uninstall openssl @0.9.8
```

You can verify whether you have the right version using the  `openssl version` command from the Terminal.

## `brew` refusing to link `openssl`

```console
Warning: Refusing to link: openssl
Linking keg-only OpenSSL means you may end up linking against the insecure,
deprecated system version while using the headers from the Homebrew version.
Instead, pass the full include/library paths to your compiler e.g.:
  -I/usr/local/opt/openssl/include -L/usr/local/opt/openssl/lib
```
This is due to a recent update from `brew` where it refuses to link `openssl`. The installation steps have been updated with instructions on how to deal with this. 

## Running .NET Core CLI on Nano Server

If you’re using Nano Server Technical Preview 5 with .NET Core CLI, due to a bug in the Nano Server product, you will need to copy binaries from  `c:\windows\system32\forwarders`. The destination depends on the [type of deployment](https://dotnet.github.io/docs/core-concepts/app-types.html) that you are choosing for your application. 

For portable applications, the forwarders need to be copied to the shared runtime location. The shared runtime can be found wherever you installed .NET Core (by default, it is `C:\Program Files\dotnet`) under the following path: `shared\Microsoft.NETCore.App\<version>\`. 

For self-contained applications, the forwarders need to be copied over into the application folder, that is, wherever you put the published output.

This process will ensure that the `dotnet` host finds the appropriate APIs it needs.  If your Nano Server Technical Preview 5 build is updated or serviced, please make sure to repeat this process, in case any of the forwarders have been updated as well.
 
Apologies for any inconvenience. Again, this has been fixed in later releases.

## Users of zsh (z shell) don't get `dotnet` on the path after install
There is a known issue in oh-my-zsh installer that interferes with how `path_helper` works on OS X systems. In short, 
the said installer creates a `.zshrc` file which contains the exploded path at the time of installation. This clobbers 
any dynamically generated path, such as the one generated by `path_helper`. 

There is an [outstanding PR](https://github.com/robbyrussell/oh-my-zsh/pull/4925) on the oh-my-zsh repo for this. 

**Workaround 1:** symlink the `dotnet` binary in the installation directory to a place in the global path, e.g. `/usr/local/bin`. 
The command you can use is:

```console
ln -s /usr/local/share/dotnet/dotnet /usr/local/bin
```

**Workaround 2:** edit your `.zshrc` and/or `.zshprofile` files to add the `/usr/local/share/dotnet` to the $PATH. 

## `app.config` file needs to be checked out before publishing 
If you have an `app.config` file in source control that places locks on local files (such as TFS), you will recieve the following error during publishing:

```console
Failed to make the following project runnable: <project name> reason: Access to the path <path> is denied.
```

To resolve this, checkout the `app.config` file from the source control prior to publishing. 

## `dotnet` commands in the root of the file system fails
If you run any `dotnet` command on project and code files that reside in the root of the file system (`/` in Linux/macOS or `C:\` in Windows) it may fail due to security reasons. The most common error that is encountered is:

> Object reference not set to an instance of an object.

This affects the situation where the actual code files are in the root. So, the example path that would trigger this behavior would be `/project.json` or `C:\project.json` on UNIX or Windows respectivelly. 

**Workaround:** use a directory to store your projects and source files. 

**More information:** https://github.com/NuGet/Home/issues/3038

## On dev builds of the tools, restoring default project from dotnet new fails
When using non-release versions of the CLI, `dotnet restore` will fail to restore `Microsoft.NETCore.App` because for that particular version it exists on a NuGet feed that is not configured on the machine. This behavior is by design and does not happen with public releases (such as RC2).

**Workaround:** create a `NuGet.config` file in the project directory which contains the following:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <!--To inherit the global NuGet package sources remove the <clear/> line below -->
    <clear />
    <add key="dotnet-core" value="https://dotnet.myget.org/F/dotnet-core/api/v3/index.json" />
    <add key="api.nuget.org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
</configuration>
```

## Running `dotnet` on Debian distributions causes a segmentation fault
If a Debian machine is set in a certain way it may cause the native host (`dotnet`) to produce a segmentation fault. The culprit is the failed installation of the `libicu` dependency due to mirror package repository setup. If you fail to set up mirror package repositories, `apt-get` may not be able to resolve the dependency and the host will fail at runtime. 

**Affects:** the native host

**Workaround:** make sure that all of the [native pre-requisites](../Documentation/prereqs.md) are installed correctly. You can usually do this by running `apt-get` package manager. 

## Uninstalling/reinstalling the PKG on OS X
OS X doesn't really have an uninstall capacity for PKGs like Windows has for 
MSIs. There is, however, a way to remove the bits as well as the "recipe" for 
dotnet. More information can be found on [this SuperUser question](http://superuser.com/questions/36567/how-do-i-uninstall-any-apple-pkg-package-file).

# Preview 3 tooling known issues

## Installing VS 2017 RC or dotnet CLI preview3 prevents Visual Studio 2015 .NET Core tooling from working
Symptoms:
Opening a project.json solution in Visual Studio 2015 fails with `error: Error reading '{some root path}\.vs\restore.dg'`

Workaround:
- Copy the full command line that is failing, something like C:\Program Files\dotnet\dotnet.exe restore "C:\Users\user\Desktop\SampleApp\.vs\restore.dg"
- Open a command prompt and navigate to the project directory, e.g. cd C:\Users\user\Desktop\SampleApp\
- Run the copied command from this directory
- Reload the project in Visual Studio

## Restore required before using the .NET Core tooling Preview 3
You have to run `dotnet restore` before you try any of the CLI commands in the Preview 3 tooling. The restore call is needed to bring in the needed targets that comprise main functionality of the Preview 3 tooling. You will get the following error if
you don't restore on macOS/Linux machines:

>  /usr/local/share/dotnet/sdk/1.0.0-preview3-004056/Microsoft.Common.CurrentVersion.targets(1107,5): error MSB3644: The reference assemblies for framework ".NETFramework,Version=v4.0" were not found. To resolve this, install the SDK or Targeting Pack for this framework version or retarget your application to a version of the framework for which you have the SDK or Targeting Pack installed. Note that assemblies will be resolved from the Global Assembly Cache (GAC) and will be used in place of reference assemblies. Therefore your assembly may not be correctly targeted for the framework you intend.

If you don't get this error on a Windows machine, that is most likely due to the fact that the targeting pack for .NET Framework 4.0 are installed. To be sure that you have the targets, run restore again if you are not sure. 

## `dotnet test` has changed from Preview 2 `dotnet test`
As part of the overall Preview 3 work, `dotnet test` command has been been revised and is quite different in usage and behavior then Preview 2 `dotnet test` command. Please consult the official [dotnet test
docs](https://docs.microsoft.com/en-us/dotnet/articles/cli-preview3/tools/dotnet-test) for more information and expect more documentation in coming days. 

# What is this document about? 
This document outlines the known issues and workarounds for the current state of 
the CLI tools. Issues will also have a workaround and affects sections if necessary. You can use this page to 
get information and get unblocked.

# What is a "known issue"?
A "known issue" is a major issue that block users in doing their everyday tasks and that affect all or 
most of the commands in the CLI tools. If you want to report or see minor issues, you can use the [issues list](https://github.com/dotnet/cli/issues). 
