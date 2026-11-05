# Mautic on Wodby

What Wodby sets up for Mautic on this service.

## Image

The service runs the prebuilt [wodby/mautic](https://github.com/wodby/mautic) image: Mautic's recommended Composer
project with its assets built, on the Apache variant of wodby/php. Apache serves Mautic and its `.htaccess` rules and
passes PHP to PHP-FPM in the same container, on port 80. There is no build and no Git repository; a new image revision
of the selected Mautic version brings Mautic's updates.

Plugins and themes can't be added to the image. Code changes to Mautic need a custom PHP application instead.

## Settings

- **Database.** `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USERNAME` and `DB_PASSWORD` from the `db` link. Mautic supports
  MariaDB and MySQL only, and can't connect to a database server over TLS.
- **Address.** Once Mautic is installed, its site URL is the app's main route (`WODBY_PRIMARY_URL`), or
  `MAUTIC_SITE_URL` when set. Links in email and cron output use it. A new main route takes effect when the service is
  redeployed; the site URL saved in Mautic's settings is ignored.
- **Proxies.** The forwarded headers of proxies on private networks are trusted (`MAUTIC_TRUSTED_PROXIES`, default
  `PRIVATE_SUBNETS`), because TLS ends before the request reaches the container.

## Data

The `files` volume is mounted at `/mnt/files` and shared by every replica and both workers through the linked storage
service. It holds Mautic's settings file (`config/local.php`), uploaded files and images, language packs, CSV imports,
logs and PHP sessions. Mautic's cache stays in each container. The database holds everything else.

`config/local.php` holds Mautic's secret key and every setting saved in Mautic. Losing it makes the database unusable to
Mautic: back up the volume with the database.

## Installation and upgrades

Mautic's web installer is not used, and the container doesn't serve it. After every deployment the **Install or
upgrade Mautic** action runs `mautic-update`:

- **First deployment:** it installs Mautic into the empty database. The administrator comes from the service's settings:
  `admin-username` (default `admin`, `MAUTIC_ADMIN_USERNAME`), `admin-password` (required, secret, at least 6
  characters, `MAUTIC_ADMIN_PASSWORD`) and `admin-email` (required, `MAUTIC_ADMIN_EMAIL`).
- Email is sent through the linked SMTP server (`sendmail` link), from the admin email and with the app's title as the
  sender name. Email and page hits are queued in the database for the queue worker. All of these can be changed in
  Mautic's settings later.
- Without an admin password or email, the action fails and installs nothing.
- The installer never runs over existing tables: if the database has tables but Mautic is not installed, for example
  after the settings file was lost, the action fails instead.
- **Later deployments:** it runs `bin/console doctrine:migrations:migrate` and `bin/console mautic:plugins:reload`.
- Changing the admin settings after the first deployment does not change the account; change it in Mautic.

## Workers

- `mautic-cron` runs Mautic's cron jobs one after another and starts again five minutes after the previous start:
  `mautic:segments:update`, `mautic:campaigns:update`, `mautic:campaigns:trigger`, `mautic:messages:send`,
  `mautic:broadcasts:send`, `mautic:import`, `mautic:contacts:scheduled_export`, `mautic:webhooks:process` and
  `mautic:reports:scheduler`. A job that fails is logged and the next one runs. `MAUTIC_CRON_INTERVAL` and
  `MAUTIC_CRON_COMMANDS` change the interval and the jobs.
- `mautic-worker` runs `bin/console messenger:consume email hit`; each run stops after an hour or 256 MB and the next
  one follows.
- Both wait until Mautic is installed. On stop, the current job or message finishes first.

## PHP settings

PHP-FPM runs as `wodby`, the owner of the code, like the console commands, so files any of them write stay writable by
the others. `max_execution_time` is 300 seconds and uploads are limited to 64 MB; Mautic's own asset limit is 6 MB until
you change it.

## Limits

- Mautic 7 on PHP 8.4. It needs MariaDB 10.11 or later or MySQL 8.4 or later.
- No development workspace.

## Check the result

- **Settings > System info** shows Mautic's own checks, its folders and its logs.
- `php bin/console doctrine:migrations:status` shows whether the database is up to date.
- The deployment's post-deploy log shows the install or upgrade output.
- The `mautic-cron` and `mautic-worker` logs show each job and queue run.
