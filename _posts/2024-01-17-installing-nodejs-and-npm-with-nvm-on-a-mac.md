---
layout: post
title: "Installing Node.js and npm on a Mac"
author: Julio Trigo
date: 2024-01-27 12:15:00 +0100
modified_date: 2025-11-29 22:45:00 +0100
last_modified_at: 2025-11-29 22:45:00 +0100
permalink: /articles/installing-nodejs-and-npm-on-a-mac/
tags:
  - Node.js
  - node
  - npm
  - nvm
  - mac
  - zsh
  - nvmrc

---

Recently, I installed Node.js and npm on my Mac and thought of documenting the steps I followed
for future reference, which will be helpful the next time I have to do it.

<!--more-->

## Choosing the Node.js version manager

As we can read in the installation and setup documentation for `npm`
([Using a Node Version Manager](https://npm.github.io/installation-setup-docs/installing/using-a-node-version-manager.html)):

> There are a lot of different versions of Node out there.
These tools will help you keep track of what version you are using, and also make it easy to
install new versions and switch between them. They also make npm easier to set up :)

I decided to go with NVM (Node Version Manager), which seems to be widely adopted and it's also the
one I was familiar with.

## nvm installation

First of all we need to install `nvm` in our system by following the instructions on its
GitHub [README](https://github.com/nvm-sh/nvm#installing-and-updating) page.

I usually prefer to go with a manual installation in order to be more in control of how things are
set up. The [Git Install](https://github.com/nvm-sh/nvm#git-install) option fits this approach:

* Clone the repository and checkout the latest version:

```shell
$ cd ~/
$ git clone https://github.com/nvm-sh/nvm.git .nvm
$ cd ~/.nvm
$ git checkout v0.39.7
```

* Activate `nvm` by sourcing it from the shell:

```shell
$ . ./nvm.sh
```

* Add these lines to the `~/.zshrc` file to have it automatically sourced in interactive shells:

```shell
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```

**Note**: it assumes we’re using `zsh`.

* Verify installation:

```shell
$ nvm --version
0.39.7
```

## node / npm installation

Check that we don’t currently have any versions of Node.js installed:

```shell
$ nvm current
none
```

Now we can install different versions of Node.js / `npm` with `nvm`:

```shell
$ nvm install 16.13.0
Downloading and installing node v16.13.0...
Downloading https://nodejs.org/dist/v16.13.0/node-v16.13.0-darwin-x64.tar.xz...
################################################################################################################################ 100.0%
Computing checksum with shasum -a 256
Checksums matched!
Now using node v16.13.0 (npm v8.1.0)
Creating default alias: default -> 16.13.0 (-> v16.13.0)
```

Alternatively, we can install the latest LTS version directly:

```shell
$ nvm install --lts
```

### Verify versions

And then we can verify the versions of `node` and `npm` that are currently active:

```shell
$ nvm current
v16.13.0

$ node --version
v16.13.0

$ npm --version
8.1.0
```

### npm version

When installing Node.js, each version of `node` is bundled with a version of `npm`:
* [Previous Releases / Node.js](https://nodejs.org/en/download/releases/)
* [Node.js distributions](https://nodejs.org/dist/index.json)

## nvm usage

### List node versions

We can list all the Node.js versions we’ve got installed:

```shell
$ nvm ls
->     v16.13.0
default -> 16.13.0 (-> v16.13.0)
node -> stable (-> v16.13.0) (default)
stable -> 16.13 (-> v16.13.0) (default)
```

And all the available versions out there:

```shell
$ nvm ls-remote
```

### Switch between node versions

If we had multiple versions of Node.js installed then we could switch between them:

```shell
$ nvm use 16.13.0
Now using node v16.13.0 (npm v8.1.0)

# Use the Node.js version set as `default`
$ nvm use default
```

## .nvmrc

As explained in the [nvm documentation](https://github.com/nvm-sh/nvm#nvmrc), we can create a
`.nvmrc` file containing a Node.js version number in the project directory (or any parent
directory). Afterwards, commands like `nvm use`, `nvm install`, or `nvm exec` will use the version
specified in the `.nvmrc` file when no version is explicitly provided.

```shell
$ cd /full/path/to/my_project

$ cat .nvmrc
17.8.0

$ nvm use
Found '/full/path/to/my_project/.nvmrc' with version <17.8.0>
Now using node v17.8.0 (npm v8.5.5)
```

## Upgrading nvm

Since we’ve just cloned the `nvm` repository, we can just fetch and check out the latest release in
order to upgrade it. See the [Manual Upgrade](https://github.com/nvm-sh/nvm#manual-upgrade) notes.

***2025-11-29 - Update:***

- Add a note about how to install the latest LTS version
- How to use the `default` Node.js version
- Clarified ambiguity with `.nvmrc`
- Improvements and fixes
