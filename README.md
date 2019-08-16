# Bullseye

## Docker

## Running the Docker Container the first time

* Clone from the current Git repo
 `git clone git@github.com:willylam/bullseye-api.git`
* Copy `.env.sample` to `.env`
* Copy `src/.env.sample` to `src/.env`
* Copy `docker-compose.override.dist.yml` to `docker-compose.override.yml`
    * override gives the ability to mount your local directory into the container for active development

* (Optional) Go to docker-compose.override.yml: services > api, add port at the end<br />
    Example: port 88 local - mapping to - port 80 of the container <br />

    `ports:` <br />
      `- "88:80"`
* Add the following to /etc/hosts  
    `127.0.0.1       bullseye.localhost rocket2day.localhost enpowerme.localhost` 
* To bring the container up, run: `docker-compose -p bullseye up -d` 

* If seeing NONE application encryption key has been specified, then you would need to generate an application key for 
    <br />Access the container, run `docker exec -it bullseye_api_1 sh`,
    <br />then set application key `php artisan key:generate`

* To create databases and seeds, run `php artisan migrate:fresh --seed`

* If success, you can view this page, assuming you are mapping port 88:80 in your .env 
    <br />Home Page: `http://enpowerme.localhost:88/home#/`
    <br />`(username: dev-admin@3marketeers.com | password: secret)`
    <br />API Docs: `http://enpowerme.localhost:88/api/documentation` 

#### .override explained

The .override contains a functional mysql image to be used in local development environments. It connects
to the main applications via hostname `mysql`.

This was done to allow the ability to decouple the base application from mysql, while still giving
a fully functional container suite checked-in for developers.

* **environment:** variables enable xdebug, sets up callback for PhpStorm to listen
* **volumes:** mounts the local src/ directory for active development

### (Optional) Via `docker-sync`

Docker has a thing with macOS file system. If a repo has a large number of files, access to files
become extremely slow, including just browsing the site. `docker-sync` is the solution.

* Copy `.env.example` to `.env`
    * This is because docker-sync does not support passing `-p` to docker-compose. :/
* Instead of `docker-compose`, run `docker-sync-stack start`

#### (Optional) Installing `docker-sync`

You can install `docker-sync` via Ruby's `gem`. See [docker-sync's homepage](http://docker-sync.io/) for more info.

* `gem install docker-sync`
    * If you get permission denied error, you're using macOS ruby. Install Ruby via Homebrew to fix this: `brew install ruby`

## Setting up PhpStorm

PhpStorm has built-in functionality to connect with Docker containers and their php clis.

### CLI Interpreter

**While your container is running**: open your settings, navigate to "PHP". Click on the `...` for CLI Interpreter.

Add new: Docker, etc

* Remote: Select "Docker"
* Service: Select the php-nginx container, `richarvey/nginx-php-fpm:latest`
* **Important**: To fix docker mappings:
    * Click on the folder next to `Docker container:` - without this update, PhpStorm will mount everything in /opt
    * Edit the volume info:
        * Change Container Path to `/var/www/html`
        * Host Path: append `src/` to the path
        * "Okay" out
    * Path Mappings should now say: `<Project root>/src→/var/www/html`

### Composer

At this time, PhpUnit does not support remote-interpreter composer. In order to `update`, `install`, or `remove` packages, use the following
command in terminal:

* `docker-compose exec api composer [...]`
* -Or- `docker exec -it 3marketeers_bullseye-api_1 composer [...]`
    * This does not require you to be within the `bullseye-api` root directory

**Tip:** If you use a sync'ing service for your code, such as Google Drive, Dropbox, or iCloud, set your `vendor/` directory
to no nosync using these steps:

* Via terminal, `mv vendor vendor.nosync`
* `ln -s vendor.nosync vendor`
* Within PhpStorm, set `vendor.nosync` to Excludes:
    * right click on `vendor.nosync` -> "Mark Directory As" -> "Excluded"
        * This removes duplicate libraries for your IDE inspector for drill-downs.

### PHPUnit

Create a new PHPUnit Configuration: Top-right drop-down, select "Edit Configurations"

Add New: "PHPUnit"
* Name: Something meaningful, e.g. "phpunit"
* Select "Defined in the configuration file"
* Click the gear icon across from alternate configuration file.
    * Add new "PHPUnit by Remote Interpreter"
    * Select your Docker CLI Interpreter, i.e. `richarvey/nginx-php-fpm:latest`
    * Default Configuration File: **full path** _on the remote server_ to your phpunit.xml, i.e. `/var/www/html/phpunit.xml`
* "Okay" out of everything

Now you can click Run to run your global PHPUnit tests. Using this default, running each class individually will work
as well.

### (Optional) Setting up Docker (via docker-compose)

**Important macOS Users**: If you're using `docker-sync`, ignore this feature and start your service via `docker-sync-stack`, [see above](#via-docker-sync).

You can also have PhpStorm manage running your container -- for viewing your site. Personally, I'd use cli for this.
I'm not too impressed with PhpStorm's management of containers. But, if you want this feature:

When you first open `bullseye-api` PhpStorm will recognize a `Dockerfile` and ask you to configure it. Click this, but don't
configure that. Within this screen, you'll have an option to add a new configuration.

Add new: Docker -> Docker-compose
* Name: use your project name, i.e. `bullseye-api`
* Compose files:
    * Add `docker-compose.yml` and `docker-compose.override.yml` (in that order)
* Environment
    * Add: `COMPOSE_PROJECT_NAME` = `bullseye`

Hit save. Now you can run it -- top-right or how ever you run your commands. A Docker window will open, with a log showing
your container being built and launched.

### PhpStorm + XDEBUG

xdebug works without configuration. To get it running within PhpStorm:

1. Download browser extension for xdebug: https://confluence.jetbrains.com/display/PhpStorm/Browser+Debugging+Extensions
2. Start up xdebug via your extension
3. Turn on PhpStorm xdebug listener

The first time you run xdebug, a window pop-up asking you for path mapping. Insure it is correct,
e.g. `[..]/src/public/index.php` -> `/var/www/html/public/index.php`

[1] https://confluence.jetbrains.com/display/PhpStorm/Zero-configuration+Web+Application+Debugging+with+Xdebug+and+PhpStorm


### Recommended Update

Because we work with libraries outside of the `public/` directory, I highly recommend you modify your path mappings
to the parent root folder:

* Open PhpStorm Settings -> "Language & Frameworks" -> "PHP" -> "Servers"
* In your default server, change your path to map `src/` -> `/var/www/html`
    * Once you do this, you can safely un-map `public/`
* "Okay" out


