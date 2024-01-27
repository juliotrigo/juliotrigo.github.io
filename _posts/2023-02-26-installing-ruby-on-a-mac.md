---
layout: post
title: "Installing Ruby on a Mac"
author: Julio Trigo
date: 2023-02-28 20:45:00 +0100
last_modified_at: 2023-02-28 20:45:00 +0100
permalink: /articles/installing-ruby-on-a-mac/
tags:
  - Ruby
  - chruby
  - ruby-install

---

There are already quite a few guides and articles that explain how to install Ruby so I don't intend to create another one.
What I'm going to do is to document the steps I took to install Ruby on my Mac and summarise how I made it work.

This is a fantastic article about [the fastest and easiest way to install Ruby on a Mac](https://www.moncefbelyamani.com/how-to-install-xcode-homebrew-git-rvm-ruby-on-mac/), linked from the [Jekyll on macOS](https://jekyllrb.com/docs/installation/macos/) installation page.

This is another article that explains the benefits of using `chruby`: [why I use chruby instead of RVM or rbenv](https://stevemarshall.com/journal/why-i-use-chruby/).

The [Why You Should Never Use sudo to Install Ruby Gems](https://www.moncefbelyamani.com/why-you-should-never-use-sudo-to-install-ruby-gems/) article is also worth reading.

<!--more-->

## Install chruby and ruby-install
I’m going to use [chruby](https://github.com/postmodern/chruby) as a Ruby version manager and [ruby-install](https://github.com/postmodern/ruby-install) as a tool to install Ruby.

They can be both installed using [Homebrew](https://brew.sh/).

```shell
$ brew install chruby ruby-install
```

## Install Ruby using ruby-install

Then we can install all the Ruby versions that we need with `ruby-install`:

```shell
$ ruby-install 2.7.7
>>> Updating ruby versions ...
>>> Installing ruby 2.7.7 into /Users/<username>/.rubies/ruby-2.7.7 ...
>>> Installing dependencies for ruby 2.7.7 ...
# ...
>>> Successfully installed ruby 2.7.7 into /Users/<username>/.rubies/ruby-2.7.7

$ ruby-install 3.1.3
>>> Installing ruby 3.1.3 into /Users/<username>/.rubies/ruby-3.1.3 ...
>>> Installing dependencies for ruby 3.1.3 ...
# ...
installing bundled gem cache:       /Users/<username>/.rubies/ruby-3.1.3/lib/ruby/gems/3.1.0/cache
>>> Successfully installed ruby 3.1.3 into /Users/<username>/.rubies/ruby-3.1.3
```

## Configure chruby in the shell

The next step is to configure `chruby` in the shell:

```bash
$ echo "source $(brew --prefix)/opt/chruby/share/chruby/chruby.sh" >> ~/.zshrc
$ echo "source $(brew --prefix)/opt/chruby/share/chruby/auto.sh" >> ~/.zshrc
$ echo "chruby ruby-3.1.3" >> ~/.zshrc
```

A few notes:
* The Ruby version in `chruby ruby-3.1.3` should match the version of Ruby that we want to use,
  which should have previously been installed
* Without that third line, we’d need to manually call `chruby` every time we want to use something
  different than the system Ruby
* It assumes we’re using  `zsh`

## Check the installation

In a new terminal (once the additions to `.zshrc` have been executed),
check that we’re using the right version of Ruby, which already comes with `bundle` installed:

```shell
$ ruby -v
ruby 3.1.3p185 (2022-11-24 revision 1a6b16756e) [x86_64-darwin21]

$ bundle --version
Bundler version 2.3.26
```

At this point, we already have Ruby `3.1.3` installed and activated!

## List available Ruby versions

```shell
$ chruby
   ruby-2.7.7
 * ruby-3.1.3
```

## Switch between Ruby versions

```shell
$ ruby -v
ruby 3.1.3p185 (2022-11-24 revision 1a6b16756e) [x86_64-darwin21]
$ chruby 2.7.7
$ ruby -v
ruby 2.7.7p221 (2022-11-24 revision 168ec2b1e5) [x86_64-darwin21]
$ chruby 3.1.3
$ ruby -v
ruby 3.1.3p185 (2022-11-24 revision 1a6b16756e) [x86_64-darwin21]
```

## Set the Ruby version in the repository

`chruby` will auto-switch Ruby versions based on the presence of a `.ruby-version` file in
the folder we `cd` to.

**NOTE**: this assumes that the `auto.sh` script has been sourced in the `.zshrc` file,
as described in one of the previous sections.

It's recommended to create a `.ruby-version` file to both document the Ruby version and
to enable the auto-switch mechanism.

```shell
$ cd /full/path/to/my_project
$ echo '3.1.3' >> .ruby-version
```

**NOTE**: remember not to add any trailing newlines to this file, as mentioned
[here](https://docs.netlify.com/configure-builds/manage-dependencies/#ruby).

Verify that it's working:

```shell
$ ruby -v
ruby 2.6.10p210 (2022-04-12 revision 67958) [universal.x86_64-darwin21]
$ cd my_project
$ ruby -v
ruby 3.1.3p185 (2022-11-24 revision 1a6b16756e) [x86_64-darwin21]
```

## Install Ruby gems

We can now install gems with `gem install` or use bundler to install them:

```shell
$ bundle add webrick
$ bundle install
```

And check the RubyGems paths:

```shell
$ gem env
RubyGems Environment:
  - RUBYGEMS VERSION: 3.3.26
  - RUBY VERSION: 3.1.3 (2022-11-24 patchlevel 185) [x86_64-darwin21]
  - INSTALLATION DIRECTORY: /Users/<username>/.gem/ruby/3.1.3
  - USER INSTALLATION DIRECTORY: /Users/<username>/.gem/ruby/3.1.0
  - RUBY EXECUTABLE: /Users/<username>/.rubies/ruby-3.1.3/bin/ruby
  - EXECUTABLE DIRECTORY: /Users/<username>/.gem/ruby/3.1.3/bin
  - SPEC CACHE DIRECTORY: /Users/<username>/.local/share/gem/specs
  - SYSTEM CONFIGURATION DIRECTORY: /Users/<username>/.rubies/ruby-3.1.3/etc
# ...
```

**NOTE**: a previous step could have been to configure a location where the gems can be
installed for our project, in isolation from the rest of the other projects we might have,
but I won't be covering it in this article.
