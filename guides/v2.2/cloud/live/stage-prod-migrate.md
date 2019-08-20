---
group: cloud-guide
title: Deploy code and migrate static files and data
functional_areas:
  - Cloud
  - Deploy
---

#### Previous step:

[Prepare to deploy to Staging and Production]({{ page.baseurl }}/cloud/live/stage-prod-migrate-prereq.html)

To migrate your database and static files to Staging and Production:

- [Deploy code](#code)
- [Migrate static files](#cloud-live-migrate-static)
- [Migrate the database](#cloud-live-migrate-db)

If you encounter errors or need to make changes, complete those updates in your local environment. Push the code changes to the Integration environment.
Deploy the updated `master` branch again. See instructions in the [previous step]({{ page.baseurl }}/cloud/live/stage-prod-migrate.html).

## Deploy code to Staging and Production {#code}

You can also use the [Project Web Interface](#interface) or [SSH and CLI commands](#ssh) to deploy your code to Staging and Production.

### Deploy code with the Project Web Interface {#interface}

The Project Web Interface provides features to create, manage, and deploy code in Integration, Staging, and Production environments for Starter and Pro plans.

For Pro projects, deploy the Integration branch you created to Staging and Production:

1. [Log in](https://accounts.magento.cloud) to your project.
1. Select the Integration branch.
1. Select the **Merge** option to deploy to Staging. Complete all testing.
1. Select the Staging branch.
1. Select the **Merge** option to deploy to Production.

For Starter, deploy the development branch you created to Staging and Production:

1. [Log in](https://accounts.magento.cloud) to your project.
1. Select the prepared code branch.
1. Select the **Merge** option to deploy to Staging. Complete all testing.
1. Select the Staging branch.
1. Select the **Merge** option to deploy to Production.

![Use the merge option to deploy]({{ site.baseurl }}/common/images/cloud_project-merge.png)

### Deploy code with SSH and CLI {#ssh}

You can use the [Magento Cloud CLI commands]({{ page.baseurl }}/cloud/reference/cli-ref-topic.html) to deploy code to Starter and Pro environments.

**Prerequisites**
- [Add public SSH key to your Magento Cloud account
- Pro Plan Pro SInterface provides features to create, manage, and deploy code in Integration, Staging, and Production environments for Starter and Pro plans.

You need the SSH and Git access information for your project.

- For Starter projects, locate the SSH and Git information through the Project Web Interface.
- For Pro projects, locate the SSH and Git information through the Project Web Interface.

#### Deploy to Pro

To deploy to Pro projects, complete the following steps:

1. Log in to the project

   ```bash
   magento-cloud login
   ```

1. List your projects:

   ```bash
   magento-cloud project:list
   ```

1. Change to a project directory. For example, `cd /var/www/html/magento2`

1. List the environments in the project:

   ```bash
   magento-cloud environment:list
   ```

1. Checkout, or switch to, the Integration environment:

   ```bash
   magento-cloud environment:checkout <environment ID>
   ```

1. Pull any updated code to your local environment

   ```
   git pull origin <environment ID>

1. Create a snapshot of the environment as a backup:

   ```bash
   magento-cloud snapshot: create -e <environment ID>
   ```

1. Complete code in your local branch.

1. Add, commit, and push changes to the environment.

   ```bash
   git add -A && git commit -m "Commit message" && git push origin <branch-name>
   ```

1. Merge with the parent envirionment

   ```bash
   magento-cloud environment:merge <environment ID>

1. Use SSH to connect to your Staging or Production environment.

1. Checkout your Staging or Production branch:

   - Staging: ` checkout staging`
   - Production: `git checkout production`

1. Pull the `master` branch from Integration.

   ```bash
   git pull origin master
   ```

   You merge this code as `staging` and `production` are branches of `master`.

1. To fully update all code, then perform a push:

   ```bash
   git push origin
   ```

## Migrate static files {#cloud-live-migrate-static}

You migrate [static files](https://glossary.magento.com/static-files) from your `pub/media` directory to Staging or Production.

We recommend using the Linux remote synchronization and file transfer command [`rsync`](https://en.wikipedia.org/wiki/Rsync). The rsync utility uses an algorithm that minimizes the amount of data by moving only the portions of files that have changed; in addition, it supports compression.

We suggest using the following syntax:

```bash
rsync -azvP <source> <destination>
```

This command uses the following options:

- `a`–archive  
- `z`–compress  
- `v`–verbose  
- `P`–partial progress  

For additional options, see the [rsync man page](http://linux.die.net/man/1/rsync).

#### To migrate static files from your local machine:

Use the rsync command to copy the `pub/media` directory from your local Magento server to staging or production:

```bash
rsync -azvP local_machine/pub/media/ <environment_ssh_link@ssh.region.magento.cloud>:pub/media/
```

#### To migrate static files from remote-to-remote environments directly (fast approach):

{:.bs-callout-info}
To transfer media from remote-to-remote environments directly, you must enable ssh agent forwarding, see [GitHub guidance](https://developer.github.com/v3/guides/using-ssh-agent-forwarding/).

1. [Open an SSH connection]({{page.baseurl}}/cloud/env/environments-ssh.html#ssh) to the source environment.

   You can find the **SSH access** link in your Project Web Interface by selecting the environment branch, and click **Access Site**:

    ```bash
    ssh -A <environment_ssh_link@ssh.region.magento.cloud>
    ```

2. Use the `rsync` command to copy the `pub/media` directory from your current environment to  another remote environment:

   ```bash
   rsync -azvP pub/media/ <destination_environment_ssh_link@ssh.region.magento.cloud>:pub/media/
   ```

## Migrate the database {#cloud-live-migrate-db}

**Prerequisite:** A database dump (see Step 3) should include database triggers. For dumping them, confirm you have the [TRIGGER privilege](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html#priv_trigger).

**Important:** The Integration environment database is strictly for development testing and may include data you may not want to migrate into Staging and Production.

For continuous integration deployments, we **do not recommend** migrating data from Integration to Staging and Production. You could pass testing data or overwrite important data. Any vital configurations will be passed using the [configuration file]({{ page.baseurl }}/cloud/live/sens-data-over.html) and `setup:upgrade` command during build and deploy.

We **do recommend** migrating data from Production into Staging to fully test your site and store(s) in a near-production environment with all services and settings.

{:.bs-callout-info }
To transfer media from remote-to-remote environments directly you must enable ssh agent forwarding, see [GitHub guidance](https://developer.github.com/v3/guides/using-ssh-agent-forwarding/)

To migrate a database:

1. SSH into the environment you want to create a database dump from:

   ```bash
   ssh -A <environment_ssh_link@ssh.region.magento.cloud>
   ```

1. List the environment relationships to find the database login information:

   ```bash
   php -r 'print_r(json_decode(base64_decode($_ENV["MAGENTO_CLOUD_RELATIONSHIPS"]))->database);'
   ```

1. Create a database dump. The following command creates a database dump as a gzip file.

   For Starter environments and Pro Integration environments:

   ```bash
   mysqldump -h <database host> --user=<database username> --password=<password> --single-transaction --triggers main | gzip - > /tmp/database.sql.gz
   ```

   For Pro Staging and Production environments, the name of the database is in the `MAGENTO_CLOUD_RELATIONSHIPS` variable (typically the same as the application name and username):

   ```bash
   mysqldump -h <database host> --user=<database username> --password=<password> --single-transaction --triggers <database name> | gzip - > /tmp/database.sql.gz
   ```

1. Transfer the database dump to another remote environment with an `rsync` command:

   ```bash
   rsync -azvP /tmp/database.sql.gz <destination_environment_ssh_link@ssh.region.magento.cloud>:/tmp
   ```

1. Enter `exit` to terminate the SSH connection.

1. Open an SSH connection to the environment you want to migrate the database into:

   ```bash
   ssh -A <destination_environment_ssh_link@ssh.region.magento.cloud>
   ```

1. Import the database dump with the following command:

   ```bash
   zcat /tmp/database.sql.gz | mysql -h <database_host> -u <username> -p<password> <database name>
   ```

   The following is an example using information from step 2:

   ```bash
   zcat /tmp/database.sql.gz | mysql -h database.internal -u user main
   ```

### Troubleshooting the database migration

If you encounter the following error, you can try to create a database dump with the DEFINER replaced:

```terminal
ERROR 1277 (42000) at line <number>: Access denied; you need (at least one of) the SUPER privilege(s) for this operation
```
{: .no-copy}

This error occurs because the DEFINER for the triggers in the SQL dump is the production user. This user requires administrative permissions.

To solve this problem, you can generate a new database dump changing or removing the `DEFINER` clause. The following is one example of completing this change:

```bash
mysqldump -h <database host> --user=<database username> --password=<password> --single-transaction main  | sed -e 's/DEFINER[ ]*=[ ]*[^*]*\*/\*/' | gzip > /tmp/database_no-definer.sql.gz
```

Use the database dump you just created to [migrate the database](#cloud-live-migrate-db).

{:.bs-callout-info}
After migrating the database, you can set up your stored procedures or views in Staging or Production the same way you did in your Integration environment.

#### Next step

[Test deployment]({{ page.baseurl }}/cloud/live/stage-prod-test.html)
