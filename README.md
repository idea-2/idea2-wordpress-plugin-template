# WordPress plugin or theme development with Visual Studio Code

This is an example repo for how one might wire up Docker Compose for local
plugin or theme development. It provides Visual Studio Code Setup, Linter, WordPress, MariaDB, WP-CLI, PHPUnit,
and the WordPress unit testing suite.

## Set up

1. User Template or Clone this repo.

2. Add `plugin.local` to `/etc/hosts`, e.g.:

    ```
    127.0.0.1 plugin.local
    ```

## Start environment

```sh
docker-compose up -d
```

The first time you run this, it will take a few minutes to pull in the required
images. On subsequent runs, it should take less than 30 seconds before you can
connect to WordPress in your browser. (Most of this time is waiting for MariaDB
to be ready to accept connections.)

The `-d` flag backgrounds the process and log output. To view logs for a
specific container, use `docker-compose logs [container]`, e.g.:

```sh
docker-compose logs wordpress
```

Please refer to the [Docker Compose documentation][docker-compose] for more
information about starting, stopping, and interacting with your environment.

## Install WordPress

navigate to `http://plugin.local/` and manually perform the famous five-second install.

Alternatively, you can use the wp-cli

```sh
docker-compose run --rm wp-cli install-wp
```

Log in to `http://plugin.local/wp-admin/` with `wordpress` / `wordpress`.

## WP-CLI

You will probably want to [create a shell alias][3] for this:

```sh
docker-compose run --rm wp-cli wp [command]
```

For example on mac:

```
# ~/.bash_profile

# alias for the WP-CLI in combination with Docker (https://github.com/idea-2/wp-plugin-theme-template)
alias wp='docker-compose run --rm wp-cli wp'
```

Reload config

```
source ~/.bash_profile
```

## Update WordPress

```sh
wp core update && wp plugin update --all
```

or use full command without alias

```sh

docker-compose run --rm wp-cli wp core update

```

## Setup New Plugin

Create new Plugin with WP-CLI:

```sh
docker-compose run --rm wp-cli wp scaffold plugin my-plugin
```

Adapt .gitignore to allow plugin (not necessary if plugin name starts with 'gbo-')

```
!plugins/my-plugin
```

Remove not used phpcs file: 'my-plugin/.phpcs.xml.dist'. There is a general config file in the plugin root folder.

Check plugin prefix for global functions

```xml
<rule ref="WordPress.NamingConventions.PrefixAllGlobals">
		<properties>
			<!-- Value: replace the function, class, and variable prefixes used. Separate multiple prefixes with a comma. -->
			<property name="prefixes" type="array" value="gbo"/>
		</properties>
	</rule>
	<rule ref="WordPress.WP.I18n">
		<properties>
			<!-- Value: replace the text domain used. -->
			<property name="text_domain" type="array" value="gbo"/>
		</properties>
	</rule>
```

## Install Dependencies

Install dependencies with Composer: `php composer.phar install`

## Running tests (PHPUnit)

The tests in this example repo were generated with WP-CLI, e.g.:

```sh
docker-compose run --rm wp-cli wp scaffold plugin-tests my-plugin
```

This is not required, however, and you can bring your own test scaffold. The
important thing is that you provide a script to install your test dependencies,
and that these dependencies are staged in `/tmp`.

The testing environment is provided by a separate Docker Compose file
(`docker-compose.phpunit.yml`) to ensure isolation. To use it, you must first
start it, then manually run your test installation script.

Note that, in the PHPUnit container, your code is mapped to `/app`.

```sh
docker-compose -f docker-compose.yml -f docker-compose.phpunit.yml up -d
docker-compose -f docker-compose.phpunit.yml run --rm wordpress_phpunit /app/bin/install-wp-tests.sh wordpress_test root '' mysql_phpunit latest true
```

Now you are ready to run PHPUnit. Repeat this command as necessary:

```sh
docker-compose -f docker-compose.phpunit.yml run --rm wordpress_phpunit phpunit
```

## Useful commands

```
wp core check-update && wp plugin list --update=available && wp theme list --update=available
```

## More Documentation at (template is based on it)

https://github.com/chriszarate/docker-compose-wordpress

## Read More
https://make.wordpress.org/cli/handbook/shell-friends/
