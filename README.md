<p align="center">
  <img src="https://raw.githubusercontent.com/smaxtec/slate/master/source/images/logo.png" alt="Slate: API Documentation Generator" width="70">
</p>

<p align="center">Slate helps us create beautiful, intelligent, responsive API documentation for smaXtec Integration API.</p>

<p align="center"><img src="https://raw.githubusercontent.com/smaxtec/slate/master/source/images/example-screenshot.png" width=700 alt="Screenshot of smaXtec Integration API Documentation created with Slate"></p>

<p align="center"><em>The example above was created with Slate. Check it out:  <a href="https://smaxtec.github.io/slate">smaXtec Integration API</a>.</em></p>


Getting Started with Slate
------------------------------

### Prerequisites

You're going to need:

 - **Linux or macOS** — Windows may work, but is unsupported.
 - **Ruby, version 2.3.1 or newer**
 - **Bundler** — If Ruby is already installed, but the `bundle` command doesn't work, just run `gem install bundler` in a terminal.

 #### Installing Ruby

 Installing with the package manager of Ubuntu might lead to errors in further commands. These installation steps are recommended:

```shell
cd $HOME
sudo apt-get update
sudo apt-get install git-core curl zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev libffi-dev software-properties-common

git clone https://github.com/rbenv/rbenv.git ~/.rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
exec $SHELL

git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
echo 'export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"' >> ~/.bashrc
exec $SHELL

rbenv install 2.5.1
rbenv global 2.5.1
ruby -v
```

 #### Installing bundler

 The installation of bundler might also not work with `gem install bundler`. Again, a different approach is recommended:

Install the version of Bundler that is declared in the lockfile.

 ```shell
 $ cat Gemfile.lock | grep -A 1 "BUNDLED WITH"
BUNDLED WITH
   1.17.3

$ gem install bundler -v '1.17.3'
 ```


### Getting Set Up

1. Clone this repository to your hard drive with `git clone https://github.com/smaxtec/slate.git`
2. `cd slate`
3. Initialize and start Slate.

```shell
bundle install
bundle exec middleman server
```

You can now see the docs at http://localhost:4567. Whoa! That was fast!

Now that Slate is all set up on your machine, you'll probably want to learn more about [editing Slate markdown](https://github.com/lord/slate/wiki/Markdown-Syntax).


Publish Slate
--------------------

Publishing your API documentation couldn't be more simple.

 1. Commit your changes.
 2. Push the *markdown source* changes to GitHub: `git push`
 3. Run `./deploy.sh`

Done! Your changes should now be live on http://smaxtec.github.io/slate, and the main branch should be updated with your edited markdown. It can also take a moment even if it's not the first time before your content is available online.

Original Slate Documentation
--------------------

[Slate Wiki](https://github.com/lord/slate/wiki)
