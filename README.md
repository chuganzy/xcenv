[![Build Status](https://travis-ci.org/xcenv/xcenv.svg?branch=master)](https://travis-ci.org/xcenv/xcenv)

# Groom your Xcode environment with xcenv.

Use xcenv to document and manage the Xcode version for your project and system. When working on multiple projects, it is often needed to support older versions of Xcode. Managing multiple Xcodes can be a problem when executing builds on the command line.

The current solutions require setting the values with `xcode-select`, using the path to the desired Xcode.app file. Additional solutions include setting `DEVELOPER_DIR` value to the desired Xcode.app path.

While these solutions work, it requires a team to use the same Xcode.app naming scheme, which can conflict from team to team. One team may desire Xcode.app to be version 6.4 and another may want it to be latest app store build. Alternatively one team may want Xcode7.2.app and another may want Xcode\_7\_2.app. For this reason xcenv uses the version number to synchronize this required toolset for a project. 

This active documentation will help your team make sure they have the right tool to build the project. Additionaly this file can be tracked in the source code changes, and therefore will be tracked. So as your project changes Xcode versions, it can be tracked. Going back to a previous commit will warn you if it may not compile with the current tools installed.

## Table Of Contents

* [How It Works](#how-it-works)
  * [Shims](#shims)
  * [Choosing the Xcode Version](#choosing-the-xcode-version)
  * [Xcode Version](#xcode-version)
  * [Finding Xcode.app](#finding-xcode.app)
  * [Plugins](#plugins)
* [Installation](#installation)
  * [Manual Installation](#manual_installation)
* [Command Reference](#command-reference)
  * [xcenv local](#xcenv-local)
  * [xcenv global](#xcenv-global)
  * [xcenv shell](#xcenv-shell)
  * [xcenv version](#xcenv-version)
  * [xcenv rehash](#xcenv-rehash)
* [Environment Variables](#environment-variables)

## How It Works

### Shims

Shims are a script that is executed instead of the desired application to inject the desired Xcode version before the command is executed.

The shims supported will work with numerous third party tools. Some tools include [Fastlane](https://fastlane.tools/), [Cocoapods](https://cocoapods.org/), [Shenzhen](https://github.com/nomad/shenzhen), and current build scripts that use `xcodebuild`, `xcrun`, or other Xcode binaries (which also includes `git`).

### Choosing the Xcode Version

When you execute a shim, xcenv determines which Xcode version to use by reading it from the following sources, in this order:

1. If `DEVELOPER_DIR` is defined, as environment or shell variable, that value will be respected and not overriden.

2. If `XCENV_VERSION` is defined, as environemnt or shell variable, that value will be used to find the matching Xcode app bundle. This can be set using the `xcenv shell` command
 
3. The first `.xcode-version` file found by searching the current working directory and each of its parent directories until reaching the root of your filesystem. You can modify the `.xcode-version` file in the current working directory with the `xcenv local` command.

4. The global ~/.xcenv/.xcode-version file. You can modify this file using the `xcenv global` command. If the global version file is not present, xcenv assumes you want to use the "system" Xcode—i.e. whatever is returned by xcode-select -p.

### Xcode Version

Xcenv supports multiple value types for the .xcode-version file.

You can set the value to the specific version desired, which will only match the specific version.

	7.3.1

You can set the value with a regular expression. This example will match any 7.3 or 7.3.x app bundles.

	7.3+

### Finding Xcode.app

Xcenv uses Spotlight with the command `mdfind "kMDItemCFBundleIdentifier == 'com.apple.dt.Xcode'"` to search for any .app bundle with the identifier that matches Xcodes

### Plugins

Xcenv supports the ability to add commands via plugins. [More Details To Come.]

## Installation

### Basic Git Installation

To install xcenv:

	$ git clone git@github.com:xcenv/xcenv.git ~/.xcenv

Copy the following into your shell profile file:

	export PATH="$HOME/.xcenv/bin:$PATH"
	eval "$(xcenv init -)"

### Homebrew

To install xcenv

	$ brew install xcenv

Follow the instructions after installing, by copying the following to shell profile:

	eval "$(xcenv init -)"

## Command Reference

### xcenv local

Sets a local project-specific Xcode version by writing the version to a .xcode-version file in the current directory. This version can be overriden by the `DEVELOPER_DIR` environment variable and the `XCENV_VERSION` variable set by the `xcenv shell` command.

	$ xcenv local 7.3.1

When ran without a version number, `xcenv local` will output the current version if available.

	$ xcenv local
	7.3.1

To unset the local version use the `--unset` parameter.

	$ xcenv local --unset

### xcenv global

Sets a global Xcode version by writing the version to a .xcode-version file in the `XCENV_ROOT` folder. This version can be overriden by the `DEVELOPER_DIR` environment variable, the `XCENV_VERSION` variable set by the `xcenv shell` command, and the `xcenv local` command.

	$ xcenv global 7.3.1

When ran without a version number, `xcenv global` will output the current version if available.

	$ xcenv global
	7.3.1

To unset the local version use the `--unset` parameter.

	$ xcenv global --unset

### xcenv shell

Sets a shell-specific Xcode version, by creating the environment variable `XCENV_VERSION`. This version can be overriden by the `DEVELOPER_DIR`.

	$ xcenv shell 7.3.1

When ran without a version number, `xcenv shell` will output the current version if available.

	$ xcenv shell
	7.3.1

To unset the local version use the `--unset` parameter.

	$ xcenv shell --unset

### xcenv version

Displays the current active Xcode version.

	$ xcenv version
	7.3.1 set by /Users/xcenv/ProjectX/.xcode-version

### xcenv rehash

Install shims for all Xcode binaries in the /usr/bin folder. The shim files will temporarily set `DEVELOPER_DIR` before calling the real `/usr/bin/${command}`

	$ xcenv rehash

## Environment Variables

### XCENV_ROOT

If you want to change where all the shims and global settings are set for xcenv, you can use the XCENV_ROOT environment variable to do so. By default the value is set to `XCENV_ROOT="${HOME}/.xcenv"`.

If you absolutely need to store everything under Homebrew's prefix, include this in your profile:

	export XCENV_ROOT=#{var}/xcenv

### XCENV_DO_NOT_SHIM_LIST

Sometimes you don't want xcenv to shim a tool. One example is if you prefer to use the latest git from [homebrew](http://brew.sh/).

To exclude a file from being shimmed, set the XCENV_DO_NOT_SHIM_LIST to a list of space delimited filenames in your profile:

	export XCENV_DO_NOT_SHIM_LIST="git c++"
