# Personal website

A Jekyll website and blog based on the [So Simple theme](https://github.com/mmistakes/so-simple-theme/).

# Setup

To setup a local environment, follow these steps:

- Install ruby 3.2.2 or later. On Linux/OSX use [rbenv](https://github.com/rbenv/rbenv) to set it up:
```bash
# Install RVM
brew install rbenv ruby-build
echo 'if which rbenv > /dev/null; then eval "$(rbenv init -)"; fi' >> ~/.zshrc 
source ~/.zshrc

# Install the ruby itself - this will take a while
rbenv install 3.2.2

# Set it as default ruby
rbenv global 3.2.2
```

- Install the Bundler gem:
```bash
rbenv init
gem install bundler
```

- Clone and navigate to the repo.

- Install all gems:
```bash
rbenv init
bundle update && bundle install
```

-  Start the Jekyll server:
```bash
bundle exec jekyll serve --watch --incremental
```

-  Open http://localhost:4000 in your browser.

# Gotchas

Running `exec jekyll serve` provides hot swapping - i.e. code changes are automatically rebuilt and
deployed locally. However, changes to `_config.yml` require the server to be restarted.


