envirius
========

``envirius`` — universal virtual environments manager.

Idea
----

Usually this kind of tools narrowly specialized for a particular
programming language. For example:

  * [virtualenv](https://github.com/pypa/virtualenv/) for python
  * [rvm](https://github.com/wayneeseguin/rvm/) for ruby
  * [kerl](https://github.com/spawngrid/kerl/) for erlang
  * [nvm](https://github.com/creationix/nvm)/[nodeenv](https://github.com/ekalinin/nodeenv/) for node.js
  * and so on

But there are many cases when in the same environment you must have more
than one programming language. For example, to create an environment with
``python2.6`` and ``node.js 0.10.24``. This idea underlies ``envirius``.

Support for new programming languages are implemented as plug-ins (see below).

Features
--------

* clean design
* easy extensible
* test coverage

Installation
============

At first:

```bash
$ git clone https://github.com/ekalinin/envirius.git
$ cd envirius
$ make install
```

Then youd need to add into your ``.bashrc`` the following:

```bash
[ -f "$HOME/.envirius/nv" ] && . ~/.envirius/nv
```

Uninstallation
==============

Just do the following:

```bash
$ cd envirius
$ make uninstall
```

Cache and previously created environments will not be deleted.

Available plugins
=================

You can create environments for the following programming languages:

* [erlang](http://erlang.org/)
* [rust](http://rust-lang.org/)
* [elixir](http://elixir-lang.org/)
* [julia](http://julialang.org/)
* [node.js](http://nodejs.org/)
* [python](https://www.python.org/)
* [go](http://golang.org/)
* [haskell](http://haskell.org/)
* [scala](http://scala-lang.org/)

New languages can be added as plugins (see below).

Usage
=====

Check available plugins
-----------------------

```bash
➥ nv ls-plugins
elixir
elixir-prebuilt
erlang
go-prebuilt
haskell
julia
node
node-prebuilt
python
rust
rust-prebuilt
scala
```

Check available versions for each plugin
----------------------------------------

```bash
➥ nv ls-versions --rust --erlang
* rust:
0.1         0.2         0.3         0.4         0.5
0.6         0.7         0.8         0.9
* erlang:
R10B-0    R10B-10   R10B-1a   R10B-2    R10B-3    R10B-4
R10B-5    R10B-6    R10B-7    R10B-8    R10B-9    R11B-0
R11B-1    R11B-2    R11B-3    R11B-4    R11B-5    R12B-0
R12B-1    R12B-2    R12B-3    R12B-4    R12B-5    R13A
R13B      R13B01    R13B02-1  R13B02    R13B03    R13B04
R14A      R14B      R14B01    R14B02    R14B03    R14B04
R15B      R15B01    R15B02    R15B02    R15B03-1  R15B03
R16A      R16B      R16B01    R16B02    R16B03-1  R16B03
17.0-rc1
```

Create an environment
---------------------

```bash
➥ nv mk mixed-rust-erlang --rust=0.9 --erlang=17.0-rc1
Creating environment: mixed-rust-erlang ...
 * installing rust==0.9 ...
 * done (in 5 secs.)
 * installing erlang==17.0-rc1 ...
 * done (in 11 secs.)
```

If you want to activate new environment just right after it creation, then do
the following:

```bash
➥ nv mk mixed-rust-erlang --rust=0.9 --erlang=17.0-rc1 --on
Creating environment: mixed-rust-erlang ...
....
Environment mixed-rust-erlang activated.
(mixed-rust-erlang) ➥ 
```

Activate/deactivate environment
-------------------------------

By default activating environment executes in a new shell:

```bash
➥ echo $$
112
➥ nv on mixed-rust-erlang
Environment mixed-rust-erlang activated.
(mixed-rust-erlang) ➥  echo $$
3437
```

So for exit just do ``exit``:
```bash
(mixed-rust-erlang) ➥  echo $$
3437
(mixed-rust-erlang) ➥  exit`
➥ echo $$
112
```

If you want to activate environment in the same shell do the following:

```bash
➥ echo $$
5099
➥ nv on --same-shell mixed-rust-erlang
Environment mixed-rust-erlang activated.
(mixed-rust-erlang) ➥  echo $$
5099
```

To deactivate this shell don't use ``exit``, use ``nv off``:

```bash
(mixed-rust-erlang) ➥  echo $$
5099
(mixed-rust-erlang) ➥ nv off
Environment mixed-rust-erlang was deactivated.
➥  echo $$
5099
```

Get list of environments
------------------------

```bash
➥ nv ls
Available environment(s):
mixed-rust-erlang
rust-0.9
erl-17-rc1
```

Get current activated environment
---------------------------------

```bash
(mixed-rust-erlang) ➥ nv current
mixed-rust-erlang
```

It will return empty, if environment is not activated:

```bash
➥ nv current
➥ 
```

Do something in environment without enabling it
-----------------------------------------------

```bash
➥ nv do node-0.10.26 'npm -g ls'
```

Get help
--------

```bash
➥ nv --help
```

Get help for a command
----------------------

For example, for ``do`` command:

```bash
➥ nv do --help
```

How to add a plugin?
====================

All plugins are in the directory
[nv-plugins](https://github.com/ekalinin/envirius/tree/master/src/nv-plugins).
If you need to add support for a new language you should add it as plugin
inside this directory.

Mandatory elements
-------------------

In the simplest case you need to implement 2 functions in the plugin's
body:

### plug_list_versions

This function should return list of available versions of the plugin.
For example:

```bash
plug_list_versions() {
    local versions=$(curl -s "http://nodejs.org/dist" | \
                grep -v "node-" | \
                egrep -o '[0-9]+\.[0-9]+\.[0-9]+' | \
                sort -u -k 1,1n -k 2,2n -k 3,3n -t .)
    echo $versions
}
```

### plug_url_for_download

This function should return full url for downloading tarball.
For example:

```bash
plug_url_for_download() {
    local version=$1
    echo "http://nodejs.org/dist/v${version}/node-v${version}.tar.gz"
}
```

Typical language installation listed in
[plug_install_default](https://github.com/ekalinin/envirius/blob/master/src/nv-commands/mk#L155)
function in ``mk`` command.

If installation is not typical then you should implement ``plug_install``
function with yourself. For example: 
[julia](https://github.com/ekalinin/envirius/blob/master/src/nv-plugins/julia).

Optional elements
-----------------

### Variables

* ``plug_list_versions_columns_count`` — number of the columns in the
  output of the ``nv ls-version`` for each plugin
* ``plug_list_versions_columns_size`` — each column width in chars in
  the output of the ``nv ls-version`` for each plugin
* ``plug_state`` — if == ``disabled`` then plugin is not active and will not
  be account in the commands:
  * mk
  * ls-versions
  * ls-plugins

### Functions

In execute order:

* ``plug_check_deps`` — check dependencies before plugin building
* ``plug_install``  — overrides the whole installation process
* ``plug_download`` — overrides default downloading sources (archive)
* ``plug_unpack`` — overrides default ``tar xzf <archive-with-source>``
* ``plug_configure`` — overrides default ``configure --prefix=<path-to-env>``
* ``plug_build`` — overrides default ``make && make install
* ``plug_build_env`` — overrides copying binaries into new environment
* ``plug_post_install_actions`` — executes after installation. For example
  ``pip`` installation in the
  [python](https://github.com/ekalinin/envirius/blob/master/src/nv-plugins/python)
  plugin

Examples
--------

* Simple: [erlang](https://github.com/ekalinin/envirius/blob/master/src/nv-plugins/erlang)
* Own impementation: [go](https://github.com/ekalinin/envirius/blob/master/src/nv-plugins/go)

Example of the usage
====================

Here is an example of building [hugo](https://github.com/spf13/hugo) static
site generator under envirius with ``go`` plugin:

```bash
$ whereis go
go:
$ nv mk go-hugo-test --go=1.2.1
Creating environment: go-hugo-test ...
 * installing go==1.2.1 ...
 * done (in 8 secs.)
$ nv ls 
Available environment(s):
go-hugo-test (go==1.2.1)
$ nv on go-hugo-test
(go-hugo-test) $ go get github.com/spf13/hugo
(go-hugo-test) $ cd $GOPATH/src/github.com/spf13/hugo
(go-hugo-test) $ go build -o hugo main.go
(go-hugo-test) $ hugo version
Hugo Static Site Generator v0.11-dev
```

Dependencies
============

* bash / zsh (very basic support)
* curl / wget
* tar
* git
* gcc
* bats

Supported OS
============

* ubuntu

Tests
=====

Most of the code is covered by tests. For testing
[bats](https://github.com/sstephenson/bats) was used.
To run tests:

```bash
➥ make tests
```

With hot cache on ``Intel(R) Core(TM) i7-3517U CPU @ 1.90GHz`` with 4 GB of RAM
on Ubuntu 13.10 (32-bits) tests takes:

```bash
➥ times make tests
...

100 tests, 0 failure

real    8m26.572s
user    1m17.428s
sys     2m25.352s
```

Version History
===============

See [CHANGELOG.md](https://github.com/ekalinin/envirius/blob/master/CHANGELOG.md).

License
=======

See [LICENSE](https://github.com/ekalinin/envirius/blob/master/LICENSE).
