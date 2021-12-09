---
title: Migrate to Drupal 9 on Pantheon
subtitle: Migrate a Drupal 9 Site to Pantheon
description: Migrate a Composer-managed Drupal 9 Site from another platform to Pantheon.
categories: [develop]
cms: drupal-9
tags: [code, launch, migrate, site, updates]
reviewed: "2021-11-12"
layout: guide
showtoc: true
permalink: docs/guides/drupal-9-migration/drupal-9-to-pantheon
anchorid: drupal-9-migration/drupal-9-to-pantheon
editpath: drupal-9-migration/05-drupal-9-to-pantheon.md
---

This doc shows how to migrate an existing Composer-managed Drupal 9 site from another platform to a new Drupal 9 site with [Integrated Composer](/integrated-composer) on Pantheon.

## Overview

In this migration you'll do the following:

- Using Git, clone your existing site locally
- Create a new Drupal 9 development site on Pantheon, and clone it locally

Working locally across both the existing and new site codebases, you'll:

- Copy selected portions of the existing site code into the new site code
- Copy portions of `composer.json` from the existing site to the new site
- Copy and commit the existing site's custom modules, themes, profiles, and other dependencies to the new site
- Make any needed customizations to `settings.php` on the new site
- Carry over any additional code customizations from the existing site to the new Pantheon Drupal 9 site code

<Alert title="Integrated Composer and existing files" type="danger">

Integrated Composer will break if `composer install` tries to modify any files that are committed in Git. To avoid this, [DO THIS THING *****]

</Alert>

## Important Notes

- Integrated Composer sites require a [nested docroot](/nested-docroot) architecture. When copying over code from the existing site, be sure to retain the new site's nested docroot structure and `web` docroot name.
  - Address the question of Composer packages being committed to version control, and how we need to not do that for IC
- Partial `drupal-9/prepare-local-environment.md` says to install the terminus site clone plugin, but I'm not sure this is used?
  - Edward to adjust to alias or `former-platform`
- `composer show` - not using this
- have them remove packages from their list that we already include (drupal/core-recommended, drupal/core-composer-scaffold..)? wikimedia merge plugin?

## Will This Guide Work for Your Site?

Before you continue, confirm that your site meets the following requirements:

- The existing Drupal 9 site uses Composer to manage site dependencies
- Composer-managed
- Able to get a local copy of the existing site / access to a Git repository of it?

## Create a New Drupal 9 Site

1. Log in to your Pantheon account. If you don't have an account yet, [create one first](https://pantheon.io/register?docs) and familiarize yourself with the [User Dashboard](/guides/quickstart/user-dashboard) before you create a new site.

1. Set up [SSH Keys](/ssh-keys) on your local computer and Pantheon account.

1. Navigate to your User Dashboard and click the **Create New Site** button:

  ...

1. Click **Visit your Pantheon Site Dashboard**

Now that you have a new site on Pantheon, you're ready to add the major components from your existing site: custom code, files, and the database.

## Prepare the Local Environment

<Partial file="drupal-9/prepare-local-environment.md" />

Create a new folder to use while working on the migration. This folder will contain two subdirectories that you'll create in the next sections, one for the site on the former platform, and one for the Pantheon site.

This doc uses the following aliases:

- **Alias:** `SITE`
- **Site Name:** `anita-drupal`
- **Working folder** ?
- **Old site folder** `FORMER-PLATFORM`
- **Pantheon site folder** `PANTHEON-D9`

### Create a Local Copy of the Old Site's Code

1. Obtain a local copy of your old site's code.  Your **code** is all custom and contributed modules or plugins, themes, and libraries. The codebase should not include the `sites/default/files` directory, or any other static assets you do not want tracked by version control.

1. Export the database and media files (`sites/default/files`) from the old platform, but do not add them or upload any files to Pantheon.

### Retrieve a Local Copy of the Pantheon Site's Code

1. From the **<span class="glyphicons glyphicons-wrench"></span> Dev** environment of the Site Dashboard, set the site's Development Mode to Git:

  ![Git connection mode](../../../images/dashboard/connection-mode-git.png)

1. Copy the `git clone` command for the site repository.

  The command should look similar to the following:

  ```bash
  git clone ssh://codeserver.dev.{site-id}@codeserver.dev.{site-id}.drush.in:2222/~/repository.git
  ```

1. Run the `git clone` command inside your working folder

<!--
### Site Structure

<Partial file="ic-upstream-structure.md" />
-->

## Add in the Custom and Contrib Code Needed to Run Your Site

What makes your site code unique is your selection of contributed modules and themes, and any custom modules or themes your development team has created. These customizations need to be replicated in your new project structure.

### Composer packages

1. Copy your package list from the "requires" section of your site's composer.json and add it to the new site's composer.json. If your composer.json defines additional repositories or patches, copy those over too. Take care not to overwrite the `upstream-configuration` package & repository.

  Note: if your old site has custom patches in its codebase, make sure to copy those over as well.

1. `composer update`

1. `git status` to see if files have been added that aren't ignored by `.gitignore`. If anything shows up other than `composer.*`, add it to `.gitignore` until `git status` only shows the composer files being modified.

1. `git add composer.*; git commit -m "Add composer packages"`

  <Accordion title="Repositories and patches in composer.json" id="repositories-and-patches-in-composer-json" icon="info-sign">

  Repository in old site's `composer.json`

  ![repository in old composer.json](https://i.imgur.com/hO0snBW.png)

  Copied to new composer.json without disturbing the "upstream-configuration" one that was already there:

  ![repository moved to new composer.json](https://i.imgur.com/T6eNnXj.png)

  Patches:

  ![repository moved to new composer.json](https://i.imgur.com/x2SYPb1.png)

  </Accordion>

### Custom Code

Manually copy custom code from the old site to the corresponding Pantheon site directory.

- `web/modules/custom` and `web/libraries` are ignored, so `git add -f`

#### Modules and Themes

[Pantheon site folder] Modules:

```bash{promptUser:user}
cp -R ../old-site/modules/custom web/modules
git add web/modules/custom
git commit -m "Copy custom modules"
```

[Pantheon site folder] Themes:

```bash{promptUser:user}
cp -R ../old-site/themes/custom web/themes
git add web/themes/custom
git commit -m "Copy custom themes"
```

Follow suit with any other custom code you need to carry over.

#### settings.php

Your existing site may have customizations to `settings.php` or other configuration files. Review these carefully and extract relevant changes from these files to copy over. Always review any file paths referenced in the code, as these paths may change in the transition.

We don't recommend that you completely overwrite the `settings.php` file with the old one, as it contains customizations for moving the configuration directory you don't want to overwrite, as well as platform-specific customizations.

The resulting `settings.php` should have no `$databases` array.

### Configuration

Copy over exported configuration from the original site. From the Pantheon D9 site, run the following commands:

  ```bash{promptUser: user}
  mkdir config
  git mv sites/default/config/* config/
  git commit -m "Add site configuration."
  ```

## Deploy

You've now committed your code additions locally. Push them up to Pantheon to deploy them to your dev environment:

  ```bash{promptUser: user}
  terminus connection:set $SITE.dev git
  git push origin master
  ```

## Add Your Database

The **Database** import requires a single `.sql` dump that contains the site's content and configurations.

1. Create a `.sql` dump using the [mysqldump](https://dev.mysql.com/doc/refman/5.7/en/mysqldump.html) utility. To reduce the size for a faster transfer, we recommend you compress the resulting archive with gzip:

  ```bash{promptUser: user}
  mysqldump -uUSERNAME -pPASSWORD DATABASENAME > ~/db.sql
  gzip ~/db.sql
  ```

   - Replace `USERNAME` with a MySQL user with permissions to access your site's database.
   - Replace `PASSWORD` with the MySQL user's password. You can also move `-p` to the end of the command and leave it blank, to be prompted for your password. This prevents your MySQL password from being visible on your terminal.
   - Replace `DATABASE` with the name of your site database within MySQL.
   - `~/db.sql` defines the output target to a file named `db.sql` in your user's home directory. Adjust to match your desired location.

  The resulting file will be named `db.sql.gz` You can use either the Pantheon Dashboard or a MySQL client to add your site's database.

1. From the Site Dashboard, select the **<span class="glyphicons glyphicons-wrench"></span> Dev** environment.

1. Select **<span class="glyphicons glyphicons-server"></span> Database / Files**.

1. Click **Import** and add your archive accordingly (based on file size):

  <TabList>

  <Tab title="Up to 100MBs" id="100mbs" active={true}>

  If your archive is under 100MB, you can upload the file directly:

   1. In the **MySQL database** field, click **File**, then **Choose File**.

   1. Select your local archive file, then press **Import**.

     ![Import MySQL database from file](../../../images/dashboard/import-mysql-file.png)

  **Note:** if you recently imported the database and need to re-import, refresh the page and use a new filename for the database file.

  </Tab>

  <Tab title="Up to 500MBs" id="500mbs">

  If your archive is less than 500MB, you can import it from URL:

   1. In the **MySQL database** field, click **URL**.

   1. Paste a publicly accessible URL for the `.sql.gz` file, and press **Import**. Change the end of Dropbox URLs from `dl=0` to `dl=1` so we can import your archive properly.

      ![Import MySQL Database from URL](../../../images/dashboard/import-mysql-url.png)

  </Tab>

  <Tab title="Over 500MBs" id="500mbsplus">

  The following instructions will allow you to add database archives larger than 500MBs using the command line MySQL client, but you can also use a GUI client like Sequel Pro or Navicat. For more information, see [Accessing MySQL Databases](/mysql-access).

   1. From the **<span class="glyphicons glyphicons-wrench"></span> Dev** environment on the Pantheon Site Dashboard, click **Connection Info** and copy the Database connection string. It will look similar to this:

    ```bash{promptUser: user}
    mysql -u pantheon -p{random-password} -h dbserver.dev.{site-id}.drush.in -P {site-port} pantheon
    ```

   1. From your terminal, `cd` into the directory containing your `.sql` file. Paste the connection string and append it with: `< database.sql`. Your command will look like:

    ```bash{promptUser: user}
    mysql -u pantheon -p{random-password} -h dbserver.dev.{site-id}.drush.in -P {site-port} pantheon < database.sql
    ```

    If you encounter a connection-related error, the DB server could be in sleep mode. To resolve this, load the site in your browser to wake it up, and try again. For more information, see [Troubleshooting MySQL Connections](/mysql-access/#troubleshooting-mysql-connections).

   1. After you run the command, the `.sql` file is imported to the **<span class="glyphicons glyphicons-wrench"></span> Dev** environment.

  </Tab>

  </TabList>

## Upload Your Files

**Files** refer to anything within `sites/default/files` for Drupal or `wp-content/uploads` for WordPress, which typically includes uploaded images, along with generated stylesheets, aggregated scripts, etc. Files are not under Git version control and are stored separately from the site's code.

You can use the Pantheon Dashboard, SFTP, or Rsync to upload your site's files.

1. Export a `tar.gz` or `.zip` file of your files directory:

  Navigate to your Drupal site's root directory to run this command, which will create an archive file in your user's home directory:

  ```bash{promptUser: user}
  cd sites/default/files
  tar -czf ~/files.tar.gz .
  ```

1. From the Site Dashboard, select the **<span class="glyphicons glyphicons-wrench"></span> Dev** environment.
1. Select **<span class="glyphicons glyphicons-server"></span> Database / Files**.
1. Click **Import** and add your archive accordingly (based on file size):

  <TabList>

  <Tab title="Up to 100MBs" id="100mbsfiles-id" active={true}>

  If your archive is under 100MB, you can upload the file directly:

   1. In the **Archive of site files** field, click **File**, then **Choose File**.

   1. Select your local archive file, then press **Import**.

  </Tab>

  <Tab title="Up to 500MBs" id="500mbsfiles">

  If your archive is less than 500MB, you can import it from URL:

   1. In the **Archive of site files** field, click **URL**.

   1. Paste a publicly accessible URL for the archive, and press **Import**. Change the end of Dropbox URLs from `dl=0` to `dl=1` so we can import your archive properly.

  </Tab>

  <Tab title="Over 500MBs" id="500mbsplusfiles">

  Rsync is an excellent method for transferring a large number of files. After performing an initial rsync, subsequent jobs will only transfer the latest changes. This can help minimize the amount of time a site is in an unpredictable state (or offline) during the final step of migration, as it allows you to bring over only new content rather than re-copying every single file.

  We recommend looking into the [Terminus Rsync Plugin](https://github.com/pantheon-systems/terminus-rsync-plugin) as a helper when doing these operations, as the number of command line arguments and specifics of directory structure make it easy for human error to impact your operation.

  To sync your current directory to Pantheon:

  ```bash{promptUser: user}
  terminus rsync . my_site.dev:files
  ```

  When using Rsync manually, the script below is useful for dealing with transfers being interrupted due to connectivity issues. It uploads files to your Pantheon site's **<span class="glyphicons glyphicons-wrench"></span> Dev** environment. If an error occurs during transfer, it waits 180 seconds and picks up where it left off:

  ```bash
  ENV='dev'
  SITE='SITEID'

  read -sp "Your Pantheon Password: " PASSWORD
  if [[ -z "$PASSWORD" ]]; then
  echo "Whoops, need password"
  exit
  fi

  while [ 1 ]
  do
  sshpass -p "$PASSWORD" rsync --partial -rlvz --size-only --ipv4 --progress -e 'ssh -p 2222' ./files/* --temp-dir=../tmp/ $ENV.$SITE@appserver.$ENV.$SITE.drush.in:files/
  if [ "$?" = "0" ] ; then
  echo "rsync completed normally"
  exit
  else
  echo "Rsync failure. Backing off and retrying..."
  sleep 180
  fi
  done
  ```

  </Tab>

  </TabList>

  You should now have all three of the major components of your site imported into Pantheon. Clear your caches on the the Pantheon Dashboard, or with terminus like so:

  ```bash{promptUser: user}
  terminus drush $SITE.dev cr
  ```

## Troubleshooting

When there are problems, you can sometimes get helpful messages about what's wrong with:

  ```bash{promptUser: user}
  terminus drush $SITE.dev watchdog:show
  ```

When you make changes to fix a problem, don't forget to rebuild cache:

  ```bash{promptUser: user}
  terminus drush $SITE.dev cr
  ```

## Finish and Review

Review the site, then proceed to launch using the [Launch Essentials](/guides/launch) documentation.
