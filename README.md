# Personal website

A Jekyll website and blog based on the [So Simple theme](https://github.com/mmistakes/so-simple-theme/).

# Setup

To setup a local environment, follow these steps:

- Install ruby 2.4.1 or later. On Linux/OSX use [RVM](https://rvm.io/rvm/install) to set it up:
```bash
# Install RVM
\curl -sSL https://get.rvm.io | bash -s stable --ruby

# Install the ruby itself
rvm install 2.4.4

# Set 2.4.1 as default ruby
rvm use --default 2.4.4
```

- Install the Bundler gem:
```bash
gem install bundler
```

- Clone and navigate to the repo.

- Install all gems:
```bash
bundle update && bundle install
```

-  Start the Jekyll server:
```bash
bundle exec jekyll serve --watch
```

-  Open http://localhost:4000 in your browser.

# Gotchas

Running `exec jekyll serve` provides hot swapping - i.e. code changes are automatically rebuilt and
deployed locally. However, changes to `_config.yml` require the server to be restarted.
