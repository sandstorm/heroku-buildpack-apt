# heroku-buildpack-apt

Add support for apt-based dependencies during both compile and runtime:
* install packages from the default package repository
* install packages from a URL
* install packages from 3rd party package repositories

## Usage

This buildpack works best with [heroku-buildpack-multi](https://github.com/sandstorm/heroku-buildpack-multi) so that it can be used with your app's existing buildpacks.

Include a list of apt package names to be installed in a file named `Aptfile`

## Example

#### .buildpacks

    https://github.com/sandstorm/heroku-buildpack-apt
    https://github.com/heroku/heroku-buildpack-nodejs

#### Aptfile

    libpq-dev
    http://downloads.sourceforge.net/project/wkhtmltopdf/0.12.1/wkhtmltox-0.12.1_linux-precise-amd64.deb
    /path/to/local/package.deb
    openjdk-8-jre

#### sources.list (optional)

    deb http://ppa.launchpad.net/openjdk-r/ppa/ubuntu trusty main
    deb-src http://ppa.launchpad.net/openjdk-r/ppa/ubuntu trusty main

#### trusted.gpg.d (optional but recommended)

    gpg --keyserver hkp://keyserver.ubuntu.com:80 --recv 86F44E2A
    gpg --export 86F44E2A > trusted.gpg.d/ppa.launchpad.net.gpg

#### Gemfile (optional)

    source "https://rubygems.org"
    gem "pg"
    
### Compile with [Anvil](https://github.com/ddollar/anvil-cli)

    $ heroku plugins:install https://github.com/ddollar/heroku-build
    
    $ heroku create apt-pg-test
    
    $ heroku build . -b ddollar/multi -r 
    Checking for app files to sync... done, 2 files needed
    Uploading: 100.0%
    Launching build process... done
    Preparing app for compilation... done
    Fetching buildpack... done
    Detecting buildpack... done, Multipack
    Fetching cache... done
    Compiling app...
    =====> Downloading Buildpack: https://github.com/ddollar/heroku-buildpack-apt
    =====> Detected Framework: Apt
      Updating apt caches
      ...
      Installing libpq-dev_8.4.17-0ubuntu10.04_amd64.deb
      Installing libpq5_8.4.17-0ubuntu10.04_amd64.deb
      Writing profile script
    =====> Downloading Buildpack: https://github.com/heroku/heroku-buildpack-ruby
    =====> Detected Framework: Ruby
      Installing dependencies using Bundler version 1.3.2
      ...
    Putting cache... done
    Creating slug... done
    Uploading slug... done
    Success, slug is https://api.anvilworks.org/slugs/00000000-0000-0000-0000-0000000000.tgz

### Check out the PG library version

    $ heroku run bash -a apt-pg-test
    ~ $ irb
    irb(main):001:0> require "pg"
    => true
    irb(main):002:0> PG::version_string
    => "PG 0.15.1"

## License

MIT

## Known Bugs

### removed packages stay in cache

If a package is installed via an URL it is downloaded and cached locally.
If the package is removed from the *Aptfile* it stays in the cache and is still installed during deployment.
