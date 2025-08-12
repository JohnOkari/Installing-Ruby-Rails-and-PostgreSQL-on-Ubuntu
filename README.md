# Installing Ruby 3.1.2, Rails 7.1.5, and PostgreSQL on Ubuntu (Using Zsh)

This README provides a step-by-step guide to install Ruby version 3.1.2, Ruby on Rails version 7.1.5, and PostgreSQL on Ubuntu. This guide assumes you're using Ubuntu 20.04 or later and Zsh as your shell. All commands are intended to be run in a terminal.
 `Change the versions where appropriate`
 
## Prerequisites
- Ensure you have administrative access (sudo privileges).
- Update your system packages:
  ```zsh
  sudo apt update
  sudo apt upgrade -y
  ```

## Step 1: Install System Dependencies
Install the necessary packages for building Ruby and supporting Rails/PostgreSQL.

```zsh
sudo apt install -y git curl libssl-dev libreadline-dev zlib1g-dev autoconf bison build-essential libyaml-dev libncurses5-dev libffi-dev libgdbm-dev
```

## Step 2: Install Ruby 3.1.2 Using rbenv
Use `rbenv` to manage Ruby versions. This allows easy switching between versions.

### 2.1: Install rbenv
```zsh
curl -fsSL https://github.com/rbenv/rbenv-installer/raw/HEAD/bin/rbenv-installer | zsh
```

### 2.2: Configure rbenv in Zsh
Add the following lines to your `~/.zshrc` file:
```zsh
export PATH="$HOME/.rbenv/bin:$PATH"
eval "$(rbenv init - zsh)"
```

Apply the changes:
```zsh
source ~/.zshrc
```

### 2.3: Install ruby-build Plugin
This plugin enables compiling Ruby versions.
```zsh
git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
```

### 2.4: Install and Set Ruby 3.1.2
```zsh
rbenv install 3.1.2
rbenv global 3.1.2
```

Verify the installation:
```zsh
ruby -v  # Expected output: ruby 3.1.2...
```

## Step 3: Install Rails 7.1.5
Rails requires Node.js and Yarn for JavaScript runtime and asset management.

### 3.1: Install Node.js and Yarn
```zsh
sudo apt install -y nodejs npm
sudo npm install -g yarn
```

### 3.2: Install Rails
```zsh
gem install rails -v 7.1.5
```

Verify the installation:
```zsh
rails -v  # Expected output: Rails 7.1.5
```

## Step 4: Install and Configure PostgreSQL
PostgreSQL is a common database for Rails applications.

### 4.1: Install PostgreSQL
```zsh
sudo apt install -y postgresql postgresql-contrib libpq-dev
```

### 4.2: Start and Enable PostgreSQL Service
```zsh
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

### 4.3: Set Up a PostgreSQL User and Database
Switch to the postgres user:
```zsh
sudo -u postgres psql
```

In the PostgreSQL prompt, run these SQL commands (replace `yourusername` and `yourpassword` with your desired credentials):
```sql
CREATE ROLE yourusername WITH LOGIN PASSWORD 'yourpassword';
ALTER ROLE yourusername CREATEDB;
CREATE DATABASE yourapp_development OWNER yourusername;
\q  # Exit the prompt
```

Verify by logging in:
```zsh
psql -U yourusername -d yourapp_development -W
```
(Enter your password when prompted.)

## Step 5: Create and Configure a Sample Rails Application
Test the setup by creating a new Rails app using PostgreSQL.

### 5.1: Create a New Rails App
```zsh
rails new yourapp -d postgresql
cd yourapp
```

### 5.2: Configure Database Credentials
Edit `config/database.yml`:
```yaml
default: &default
  adapter: postgresql
  encoding: unicode
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: yourusername
  password: yourpassword
  host: localhost

development:
  <<: *default
  database: yourapp_development

test:
  <<: *default
  database: yourapp_test
```

### 5.3: Create the Database
```zsh
rails db:create
```

### 5.4: Start the Rails Server
```zsh
rails server
```

Visit `http://localhost:3000` in your browser to see the Rails welcome page.

## Troubleshooting
- **rbenv not found**: Ensure `~/.zshrc` is sourced correctly.
- **Gem permission issues**: Avoid using `sudo` with gems; rbenv handles this.
- **PostgreSQL errors**: Check service status with `sudo systemctl status postgresql`.
- **Version mismatches**: Run `rbenv rehash` after installing gems.

If you encounter issues, check the official documentation for [rbenv](https://github.com/rbenv/rbenv), [Rails](https://guides.rubyonrails.org/), or [PostgreSQL](https://www.postgresql.org/docs/).
