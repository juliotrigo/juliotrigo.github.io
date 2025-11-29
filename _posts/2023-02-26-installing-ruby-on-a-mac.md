---
layout: post
title: "Installing Ruby on a Mac"
author: Julio Trigo
date: 2023-02-28 20:45:00 +0100
modified_date: 2025-11-29 16:45:00 +0100
last_modified_at: 2025-11-29 16:45:00 +0100
permalink: /articles/installing-ruby-on-a-mac/
tags:
  - Ruby
  - chruby
  - ruby-install
  - bundle
  - ruby-version
  - gem

---

There are already quite a few guides and articles that explain how to install Ruby, so I don't
intend to write just another one.

Instead, I’m going to document the steps I followed to install Ruby on my Mac, describe the approach
I chose, and summarise the commands I used. These notes are meant to serve as my personal technical
reference and to help clarify how the Ruby installation process works.

<!--more-->

## Bibliography

This is a fantastic article about [the fastest and easiest way to install Ruby on a Mac](https://www.moncefbelyamani.com/how-to-install-xcode-homebrew-git-rvm-ruby-on-mac/), linked from the [Jekyll on macOS](https://jekyllrb.com/docs/installation/macos/) installation page.

This is another article that explains the benefits of using `chruby`: [why I use chruby instead of RVM or rbenv](https://stevemarshall.com/journal/why-i-use-chruby/).

The [Why You Should Never Use sudo to Install Ruby Gems](https://www.moncefbelyamani.com/why-you-should-never-use-sudo-to-install-ruby-gems/) article is also worth reading.

And an article about configuring Bundler to [install gems in the project directory](https://guilhermesimoes.github.io/blog/installing-gems-per-project-directory).

## Install chruby and ruby-install

I’m going to use [chruby](https://github.com/postmodern/chruby) as a Ruby version manager and [ruby-install](https://github.com/postmodern/ruby-install) as a tool to install Ruby.

They can both be installed using [Homebrew](https://brew.sh/).

```shell
$ brew install chruby ruby-install
```

## Install Ruby versions

Then we can install all the Ruby versions that we need with `ruby-install`:

```shell
$ ruby-install 3.4.7
>>> Installing ruby 3.4.7 into /Users/<username>/.rubies/ruby-3.4.7 ...
>>> Installing dependencies for ruby 3.4.7 ...
>>> Downloading https://cache.ruby-lang.org/pub/ruby/3.4/ruby-3.4.7.tar.xz into /Users/<username>/
>>> Verifying ruby-3.4.7.tar.xz ...
>>> Extracting ruby-3.4.7.tar.xz to /Users/<username>/src/ruby-3.4.7 ...
>>> Configuring ruby 3.4.7 ...
>>> Cleaning ruby 3.4.7 ...
>>> Compiling ruby 3.4.7 ...
>>> Installing ruby 3.4.7 ...
installing bundled gems:            /Users/<username>/.rubies/ruby-3.4.7/lib/ruby/gems/3.4.0
>>> Successfully installed ruby 3.4.7 into /Users/<username>/.rubies/ruby-3.4.7

$ ruby-install 3.4.5
>>> Installing ruby 3.4.5 into /Users/<username>/.rubies/ruby-3.4.5 ...
>>> Installing dependencies for ruby 3.4.5 ...
>>> Downloading https://cache.ruby-lang.org/pub/ruby/3.4/ruby-3.4.5.tar.xz into /Users/<username>/src ...
>>> Verifying ruby-3.4.5.tar.xz ...
>>> Extracting ruby-3.4.5.tar.xz to /Users/<username>/src/ruby-3.4.5 ...
>>> Configuring ruby 3.4.5 ...
>>> Cleaning ruby 3.4.5 ...
>>> Compiling ruby 3.4.5 ...
>>> Installing ruby 3.4.5 ...
installing bundled gems:            /Users/<username>/.rubies/ruby-3.4.5/lib/ruby/gems/3.4.0
>>> Successfully installed ruby 3.4.5 into /Users/<username>/.rubies/ruby-3.4.5
```

Ruby versions are installed in the `~/.rubies/` folder by default:

```shell
$ ls ~/.rubies
ruby-3.4.5 ruby-3.4.7
```

## Configure chruby in the shell

The next step is to configure `chruby` in the shell (it assumes we're using `zsh`):

```shell
# Load and enable chruby
$ echo "source $(brew --prefix)/opt/chruby/share/chruby/chruby.sh" >> ~/.zshrc

# Enable auto-switching of Rubies specified by .ruby-version files
$ echo "source $(brew --prefix)/opt/chruby/share/chruby/auto.sh" >> ~/.zshrc

# Set the default Ruby version, which should have previously been installed
# Without this, we’d need to manually call `chruby` every time we want to use
# something different than the system Ruby
$ echo "chruby ruby-3.4.7" >> ~/.zshrc
```

## Reload the shell

```shell
source ~/.zshrc
```

## Check the installation

Check that we’re using the right version of Ruby, which already comes with Bundler installed:

```shell
$ ruby -v
ruby 3.4.7 (2025-10-08 revision 7a5688e2a2) +PRISM [arm64-darwin24]

$ bundle --version
Bundler version 2.6.9
```

At this point, we already have Ruby `3.4.7` installed and activated!

## List available Ruby versions

```shell
$ chruby
   ruby-3.4.5
 * ruby-3.4.7
```

## Switch between Ruby versions

```shell
$ ruby -v
ruby 3.4.7 (2025-10-08 revision 7a5688e2a2) +PRISM [arm64-darwin24]

$ chruby 3.4.5
$ ruby -v
ruby 3.4.5 (2025-07-16 revision 20cda200d3) +PRISM [arm64-darwin24]

$ chruby 3.4.7
$ ruby -v
ruby 3.4.7 (2025-10-08 revision 7a5688e2a2) +PRISM [arm64-darwin24]
```

## Set the Ruby version in the project

It's recommended to create a `.ruby-version` file to both document the Ruby version and
enable the auto-switch mechanism.

```shell
$ cd my_project
$ echo '3.4.5' > .ruby-version
```

`chruby` will auto-switch Ruby versions based on the presence of a `.ruby-version` file in
the folder we `cd` to.

```shell
$ ruby -v
ruby 3.4.7 (2025-10-08 revision 7a5688e2a2) +PRISM [arm64-darwin24]

$ cd my_project
$ ruby -v
ruby 3.4.5 (2025-07-16 revision 20cda200d3) +PRISM [arm64-darwin24]

$ cd ..
$ ruby -v
ruby 2.6.10p210 (2022-04-12 revision 67958) [universal.arm64e-darwin24]
```

If you then `cd` into a different directory that either doesn't have a `.ruby-version`
file or if the `.ruby-version` file is set to a Ruby version that `chruby` doesn't know
about, then `chruby` will revert to the *system* Ruby on my Mac (`2.6.10` in this case).

**Notes**:
- This assumes that the `auto.sh` script has been sourced in the `.zshrc` file,
  as described in one of the [previous sections](#configure-chruby-in-the-shell).
- Remember not to add any trailing newlines to this file, as mentioned
  [here](https://docs.netlify.com/configure-builds/manage-dependencies/#ruby).

## Install Ruby gems

We can now use Bundler to install the gems:

```shell
$ bundle install
```

Bundler is the standard tool in Ruby for managing project dependencies.

### Installation location

Gems are installed in the `~/.gem` folder and segregated by Ruby minor version (`3.3`, `3.4`),
not patch version. So Ruby `3.4.5` and `3.4.7` share the same gem directory (`~/.gem/ruby/3.4.0/`).

`bundle install` normally installs gems in the same location as `gem install`. A few exceptions
(not all) are:
- If we've set up a custom `bundle` path (see [next section](#isolate-gems-per-project))
  - If `.bundle/config` exists with a path
  - The `BUNDLE_PATH` environment variable is set
- If we're using the `--deployment` flag

**Notes**:
- By default, all projects using the same Ruby version share the same gem installation directory
- Bundler doesn't provide filesystem isolation for gems by default
- Bundler isolates gems at runtime through the load path (`$LOAD_PATH`)
- Multiple versions of the same gem can coexist in the gem installation directory
- Bundler achieves isolation by manipulating the load path to include only the gems specified
  in the project's `Gemfile.lock`

### Isolate gems per project

We could also choose to use filesystem isolation per project by installing gems locally in
`vendor/bundle` and configuring Bundler to use it:

```shell
# Add the Bundle path to the Bundle config
bundle config set --local path 'vendor/bundle'

# Install the gems in the Bundle path specified in the Bundle config
bundle install

# Add these files to .gitignore
echo "vendor/bundle/" >> .gitignore
echo ".bundle/" >> .gitignore
```

The `bundle config` command above creates a `.bundle/config` file that remembers this setting:

```yaml
---
BUNDLE_PATH: "vendor/bundle"
```

Subsequent bundle install commands will install gems in the `vendor/bundle` folder.

## Check the gem environment

```shell
$ gem env
RubyGems Environment:
  - RUBYGEMS VERSION: 3.6.9
  - RUBY VERSION: 3.4.7 (2025-10-08 patchlevel 58) [arm64-darwin24]
  - INSTALLATION DIRECTORY: /Users/<username>/.gem/ruby/3.4.7
  - USER INSTALLATION DIRECTORY: /Users/<username>/.local/share/gem/ruby/3.4.0
  - CREDENTIALS FILE: /Users/<username>/.local/share/gem/credentials
  - RUBY EXECUTABLE: /Users/<username>/.rubies/ruby-3.4.7/bin/ruby
  - GIT EXECUTABLE: /usr/bin/git
  - EXECUTABLE DIRECTORY: /Users/<username>/.gem/ruby/3.4.7/bin
  - SPEC CACHE DIRECTORY: /Users/<username>/.cache/gem/specs
  - SYSTEM CONFIGURATION DIRECTORY: /Users/<username>/.rubies/ruby-3.4.7/etc
  - RUBYGEMS PLATFORMS:
     - ruby
     - arm64-darwin-24
  - GEM PATHS:
     - /Users/<username>/.gem/ruby/3.4.7
     - /Users/<username>/.rubies/ruby-3.4.7/lib/ruby/gems/3.4.0
  - GEM CONFIGURATION:
     - :update_sources => true
     - :verbose => true
     - :backtrace => true
     - :bulk_threshold => 1000
  - REMOTE SOURCES:
     - https://rubygems.org/
  - SHELL PATH:
     - /Users/<username>/.gem/ruby/3.4.7/bin
     - /Users/<username>/.rubies/ruby-3.4.7/lib/ruby/gems/3.4.0/bin
     - /Users/<username>/.rubies/ruby-3.4.7/bin
     # ...
```

## Summary

- Install Ruby using `chruby` + `ruby-install` + Bundler
- This solution provides everything I need
- Simple installation and usability
- `ruby-install` handles installing Ruby versions
- `chruby` handles switching Ruby versions (also automatically via `.ruby-version` files)
- Bundler handles gem dependencies management via `Gemfile` and `Gemfile.lock`
  - Gem isolation per Ruby version
  - Gem isolation per project at runtime: applications only load gems specified in the
    `Gemfile.lock` file

***2025-11-29 - Update:***

- All the sections are now up to date
- Ruby versions updated
- Expanded the section about installing Ruby gems
- Added a summary
