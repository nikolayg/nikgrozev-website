# Personal website

A Jekyll website and blog based on the [So Simple theme](https://mmistakes.github.io/so-simple-theme/).

# Setup

To setup a local environment, follow these steps:

1. Install ruby 2.3.1 or later. On Linux/OSX use [RVM](https://rvm.io/rvm/install) to set it up:
```bash
# Install RVM
\curl -sSL https://get.rvm.io | bash -s stable --ruby

# Install the ruby itself
rvm install 2.3.1

# Set 2.3.1 as default ruby
rvm use --default 2.3.1
```

2. Install the Bundler gem:
```bash
gem install bundler
```

3. Clone and navigate to the repo.

4. Install all gems:
```bash
bundle update && bundle install
```

5. Start the Jekyll server:
```bash
bundle exec jekyll serve --watch
```

6. Open http://localhost:4000 in your browser.

# Gotchas

Running `exec jekyll serve` provides hot swapping - i.e. code changes are automatically rebuilt and
deployed locally. However, changes to `_config.yml` require the server to be restarted.
