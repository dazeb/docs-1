# Getting Started

Pelican Panel is designed to run on your webserver.

You are expected to read through our documentation.
We have spent a lot of time curating these docs for the community,
so please take some time to read through them before asking for help on the forums!

> [!WARNING]
> You should have some basic familiarity with Linux before your proceed!

## Picking an Operating System (OS)

Pelican runs on a wide range of operating systems, so pick whichever you are most comfortable using. Note: Other OS's, not listed below, might still work. 

####  __OpenVZ, **unless specifically configured**, will **not** work with Pelican.__  

| Operating System | Version | Supported | Notes                                                       |
|:----:|:-:|:---:|:-----:|
| **Ubuntu**       | 20.04   |     ✅︎    | Ubuntu 20.04 EoL is April 2025, not recommended             |
|                  |**22.04**|     ✅︎    | Documentation written assuming Ubuntu 22.04 as the base OS. |
|                  | 24.04   |     ✅︎    |                                                             |
| **Rocky Linux**  | 9       |     ✅︎    |                                                             |
| **Debian**       | 11      |     ✅︎    |                                                             |
|                  | 12      |     ✅︎    |                                                             |

## Dependencies  

 The `ondrej/php` repository is required to install the latest versions of PHP and its required extensions.

 It can be added with the following command.
 ```sh
 sudo add-apt-repository ppa:ondrej/php
 ```

* PHP `8.2` with the following extensions: `gd`, `mysql`, `mbstring`, `bcmath`, `xml`, `curl`, `zip`, `intl` and `fpm`
* MySQL `8` (`mysql-server`) **or** MariaDB `10.2`+
* A webserver (Apache, NGINX, Caddy, etc.)
* `curl`
* `tar`
* `composer` v2

## Create Directories & Downloading Files

The first step in this process is to create the folder where the panel will live and then move ourselves into that
newly created folder.

```sh
mkdir -p /var/www/pelican
cd /var/www/pelican
```

Once you have created a new directory to use and moved into it you'll need to download the Panel files. This
is as simple as using `curl` to download our pre-packaged content. Once it is downloaded you'll need to unpack the archive
and then set the correct permissions on the `storage/` and `bootstrap/cache/` directories. These directories
allow us to store files as well as keep a speedy cache available to reduce load times.

```sh
curl -Lo panel.tar.gz https://github.com/pelican-dev/panel/releases/latest/download/panel.tar.gz
tar -xzvf panel.tar.gz
chmod -R 755 storage/* bootstrap/cache/
```

## Installation

Next we will set up the default environment settings file, dependencies, and then generate a new application encryption key.

```sh
cp .env.example .env
composer install --no-dev --optimize-autoloader
```


    Only run the command below if you are installing Pelican for the first time!  
    If you're trying to update your panel, [Go Here](./update)

    **Running this command with an existing install will cause irrecoverable dataloss!**
    ```sh
    php artisan key:generate --force
    ```


## Database Setup

You will need a database and a user with the correct permissions created for that database before continuing any further. 

## Logging Into MySQL  

The first step in this process is to login to the MySQL command line where we will be executing statements to get
things setup. To do so, simply run the command below and provide the Root MySQL account's password that you setup when
installing MySQL. If you do not remember doing this, chances are you can just hit enter as no password is set.

```sh
mysql -u root -p
```

## Creating MySQL User  

Next, we will create a user called `pelican` and allow logins from localhost which prevents any external connections
to our database. You can also use `%` as a wildcard or enter a numeric IP. We will also set the account password
to `somePassword`.

```sql
CREATE USER 'pelican'@'127.0.0.1' IDENTIFIED BY 'somePassword';
```

## Creating Database
Next, we need to create a database for the panel. In this tutorial we will be naming the database `panel`, but you can
substitute that for whatever name you wish.

```sql
CREATE DATABASE panel;
```

## Assigning Permissions
Finally, we need to tell MySQL that our pelican user should have access to the panel database. To do this, simply
run the command below.

```sql
GRANT ALL PRIVILEGES ON panel.* TO 'pelican'@'127.0.0.1';
```

## Environment Configuration

    Back up your encryption key (APP_KEY in the `.env` file). This is used as an encryption key for all data that needs to be stored securely (e.g. api keys).
    Store it somewhere safe - not just on your server. If you lose it all encrypted data is irrecoverable -- **even if you have database backups.**


The core environment is easily configured using a few different CLI commands built into the app. This step
will cover setting up things such as sessions, caching, database credentials, and email sending.

```sh
php artisan p:environment:setup
php artisan p:environment:database
php artisan p:environment:mail
```

## Database Initialization 

Now we need to set up database for the Panel that you created before.
**The command below may take some time to run depending on your machine.
Please _DO NOT_ exit the process until it is completed!**
After the database tables are created, it'll add some default Eggs to your Panel.

```sh
php artisan migrate --seed --force
```

## Creating User

You'll then need to create an administrative user so that you can log into the panel. To do so, run the command below.
At this time passwords **must** meet the following requirements: 8 characters, mixed case, at least one number.

```sh
php artisan p:user:make
```

## Crontab Configuration

We need to create a new cronjob that runs every minute to process specific tasks, such as session cleanup and scheduled tasks.
You'll want to open your crontab using `sudo crontab -e` and then paste the line below.

```sh
* * * * * php /var/www/pelican/artisan schedule:run >> /dev/null 2>&1
```

### Setting Permissions

The last step in the installation process is to set the correct permissions on the Panel files so that the webserver can
use them correctly.

## Apache Webserver

```sh
chown -R www-data:www-data /var/www/pelican/* 
```
## Nginx

```sh 
chown -R nginx:nginx /var/www/pelican/* 
```
## Rocky Linux Apache

```sh 
chown -R apache:apache /var/www/pelican/* 
```
