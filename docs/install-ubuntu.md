# Installing on Ubuntu 12.10

This installation guide was created for and tested on Ubuntu 12.10. Please read [`doc/install/requirements.md`](./requirements.md) for hardware and OS requirements.

## Overview

This guide sets up a production server running Coinpunk. The basic procedure includes:

* Installing packages
* Installing, configuring and starting bitcoind
* Installing Coinpunk
* Installing and configuring the SQL database
* Configuring Nginx



### Installing packages

First, update your package list. If your `sudo` is not present, you should run the update commands as root, and then run `apt-get install sudo`:

    sudo apt-get update
    sudo apt-get upgrade

Now you will need to install the necessary packages:

    sudo apt-get install redis-server ntp build-essential zlib1g-dev libyaml-dev libssl-dev libgdbm-dev libreadline-dev libncurses5-dev libffi-dev git-core checkinstall libxml2-dev libxslt-dev libcurl4-openssl-dev libicu-dev software-properties-common python-software-properties

Now, you need to install Ruby. There are two ways to do this. The first way is to install ruby using the package system:

    sudo apt-get install ruby

If you would rather compile ruby from source, [download the latest version of ruby](http://www.ruby-lang.org/en/downloads) and install from source:

    gzip -dc ruby-2.0.0-p195.tar.gz | tar xvf -w
    cd ruby-2.0.0-p195
    ./configure
    make
    sudo make install
    
Now you will need to install bundler, which will download and compile the needed dependencies:

    `sudo gem install bundler`

### Installing, configuring and starting bitcoind

You can install bitcoind from the bitcoin package repository:

    sudo add-apt-repository ppa:bitcoin/bitcoin
    sudo aptitude update
    sudo aptitude install bitcoind
    
Now you will need to create a `.bitcoin` folder to store your data and config for bitcoind:

    mkdir ~/.bitcoin

Create a `bitcoin.conf` file, and add the following values:

    echo "gen=0" > ~/.bitcoin/bitcoin.conf
    echo "rpcuser=YOURUSER" >> ~/.bitcoin/bitcoin.conf
    echo "rpcpassword=YOURPASSWORD" >> ~/.bitcoin/bitcoin.conf
    echo "maxconnections=5" >> ~/.bitcoin/bitcoin.conf

Start the bitcoind server:

    bitcoind --daemon
    
Bitcoind will take several hours to catch up with the bitcoin network, so please be patient.

### Installing Coinpunk

Now you will need to clone Coinpunk:

    git clone https://github.com/kyledrake/coinpunk.git
    cd coinpunk
    
Copy the `config.yml.template` to `config.yml`:

    cp config.yml config.yml.template
    
Now you will need to edit config.yml using `vi`, `nano` or a similar editor. Provide a random alpha-numeric value for `session_secret`, create a unique username and password for `bitcoind_rpcuser` and `bitcoind_rpcpassword`, and optionally add `bitcoind_rpchost` if you plan to access bitcoind from a different (non-localhost) server.

Your `config.yml` should look something this (with an optional `development` and `test` config if you're doing non-production work):

    production:
      database: 'sqlite://coinpunk.sqlite3'
      session_secret: RANDOMSECRET
      bitcoind_rpcuser: YOURRPCUSER
      bitcoind_rpcpassword: YOURRPCPASS

### Installing and configuring the SQL database

If you don't have a preference for SQL databases, you can use SQLite, which will just store data in a file in the root of your application. The config.yml is configured this way by default.

To install:

    sudo apt-get install sqlite3 libsqlite3-dev

If you would like to use MySQL or PostgreSQL, simply change the database config to point to your database. For example, with PostgreSQL, you would use `postgres://USER:PASS@localhost/coinpunk`.

Now you will need to run `bundle install` to grab the required code.

If you are just using Sqlite3 (default), run this:

    sudo bundle install --without development test postgres mysql

If you are using PostgreSQL:

    sudo bundle install --without development test sqlite mysql

If you are using MySQL:

    sudo bundle install --without development test sqlite postgres

