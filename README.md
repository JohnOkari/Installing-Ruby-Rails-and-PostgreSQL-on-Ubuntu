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

# ALTER ROLE
In PostgreSQL, altering roles (users) is done using the `ALTER ROLE` command. Based on the context of your previous question about installing Ruby, Rails, and PostgreSQL on Ubuntu, I’ll assume you’re referring to modifying the PostgreSQL role (`yourusername`) created during the setup. Below, I’ll provide a step-by-step guide to alter roles in PostgreSQL, focusing on common modifications like changing passwords, privileges, or role attributes. These steps assume you're using Zsh as your shell, as specified earlier.

### Step 1: Access the PostgreSQL Prompt
Log in to PostgreSQL as the `postgres` superuser:
```zsh
sudo -u postgres psql
```

This opens the PostgreSQL interactive prompt (`psql`).

### Step 2: Identify the Role to Alter
List all roles to confirm the role you want to modify (e.g., `yourusername`):
```sql
\du
```
This displays a table of roles and their attributes.

### Step 3: Common Role Alterations
Here are examples of common changes you might want to make to the role `yourusername`. Run these commands in the `psql` prompt.

#### 3.1: Change the Role’s Password
To update the password:
```sql
ALTER ROLE yourusername WITH PASSWORD 'newpassword';
```

#### 3.2: Grant or Revoke Database Creation Privileges
If you need to modify whether the role can create databases:
- Grant:
  ```sql
  ALTER ROLE yourusername WITH CREATEDB;
  ```
- Revoke:
  ```sql
  ALTER ROLE yourusername WITH NOCREATEDB;
  ```

#### 3.3: Grant or Revoke Superuser Privileges
To make the role a superuser (use with caution, as this gives full database access):
- Grant:
  ```sql
  ALTER ROLE yourusername WITH SUPERUSER;
  ```
- Revoke:
  ```sql
  ALTER ROLE yourusername WITH NOSUPERUSER;
  ```

#### 3.4: Allow Role to Log In
If the role can’t log in, enable it:
```sql
ALTER ROLE yourusername WITH LOGIN;
```
To disable login:
```sql
ALTER ROLE yourusername WITH NOLOGIN;
```

#### 3.5: Grant Specific Privileges on a Database
To grant the role specific permissions (e.g., on `yourapp_development`):
- Grant all privileges on a database:
  ```sql
  GRANT ALL PRIVILEGES ON DATABASE yourapp_development TO yourusername;
  ```
- Grant specific privileges (e.g., SELECT, INSERT):
  ```sql
  GRANT SELECT, INSERT ON ALL TABLES IN SCHEMA public TO yourusername;
  ```

To revoke privileges:
```sql
REVOKE ALL ON DATABASE yourapp_development FROM yourusername;
```

#### 3.6: Rename the Role
To rename the role:
```sql
ALTER ROLE yourusername RENAME TO newusername;
```
**Note**: If you rename the role, update your Rails `config/database.yml` to reflect the new username.

#### 3.7: Set Role Attributes
You can modify other attributes, like connection limits or role expiration:
- Set a connection limit:
  ```sql
  ALTER ROLE yourusername WITH CONNECTION LIMIT 100;
  ```
- Set a role to expire (e.g., on a specific date):
  ```sql
  ALTER ROLE yourusername VALID UNTIL '2026-01-01';
  ```

### Step 4: Verify Changes
Check the updated role attributes:
```sql
\du
```
This lists all roles and their updated attributes.

Exit the `psql` prompt:
```sql
\q
```
