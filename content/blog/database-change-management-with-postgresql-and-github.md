---
title: Database Change Management with PostgreSQL and GitHub
author: Ningjing
published_at: 2023/02/16 11:45
feature_image: /static/blog/database-change-management-with-postgresql-and-github/feature-image.webp
tags: Tutorial
integrations: PostgreSQL, GitHub
description: This tutorial will bring your PostgreSQL schema change to the next level by introducing the GitOps workflow, where you commit schema change script to the GitHub repository, which will in turn trigger the schema deployment pipeline in Bytebase.
---

This is a series of articles about Database Change Management with PostgreSQL

- [Database Change Management with PostgreSQL](/blog/database-change-management-with-postgresql)
- Database Change Management with PostgreSQL and GitHub (this one)

---
## Overview

In the last article [Database Change Management with PostgreSQL](/blog/database-change-management-with-postgresql), you have tried UI workflow in Bytebase.

This tutorial will bring you to the next level by introducing the GitOps workflow, where you commit the schema change script to the GitHub repository, which will in turn trigger the schema deployment pipeline in Bytebase.

You can use Bytebase free version to finish the tutorial.

## Prerequisites

Before you start this tutorial, make sure:

- You have followed our previous UI-based change tutorial [Database Change Management with PostgreSQL](/blog/database-change-management-with-postgresql).
- You have a GitHub account.
- You have a public GitHub repository, e.g  `pg-test-bb-local`.
- You have [Docker](https://www.docker.com/) installed locally.
- You have a [ngrok](http://ngrok.com) account. ngrok is a reverse proxy tunnel, and in our case, we need it for a public network address in order to receive webhooks from [GitHub.com](http://GitHub.com). We use ngrok here for demonstration purposes. For production use, we recommend using [Caddy](https://caddyserver.com/).
![ngrok](/static/blog/database-change-management-with-postgresql-and-github/ngrok.webp)

## Step 1 - Run Bytebase in Docker with URL generated by ngrok

To make local-running Bytebase visible to GitHub, we’ll pass ngrok generated URL to [--external-url](https://www.bytebase.com/docs/get-started/install/external-url).

1. Login to [ngrok Dashboard](https://dashboard.ngrok.com/) and follow its [Getting Started](https://dashboard.ngrok.com/get-started/setup) steps to install and configure.
2. Run
```bash
ngrok http 5678
```
and obtain the public URL:
![terminal-ngrok](/static/blog/database-change-management-with-postgresql-and-github/terminal-ngrok.webp)

3. Make sure your Docker daemon is running, if it’s running Bytebase container for [the previous tutorial](/blog/database-change-management-with-postgresql), stop and remove it. The data created in the last tutorial is stored under `~/.bytebase/data` by default and will be restored if the system restarts.
   
4. Start the Bytebase Docker container by typing the following command in the terminal. Pay attention to the last parameter `--external-url https://70ca-154-9-204-36.ngrok.io`, which is generated by ngrok.
```bash
docker run --init \
--name bytebase \
--restart always \
--publish 5678:8080 \
--health-cmd "curl --fail http://localhost:5678/healthz || exit 1" \
--health-interval 5m \
--health-timeout 60s \
--volume ~/.bytebase/data:/var/opt/bytebase \
bytebase/bytebase:%%bb_version%% \
--data /var/opt/bytebase \
--port 8080 \
--external-url https://70ca-154-9-204-36.ngrok.io
````

5. Bytebase is running successfully in Docker, and you can visit it via https://70ca-154-9-204-36.ngrok.io/
![docker](/static/blog/database-change-management-with-postgresql-and-github/docker.webp)

## Step 2 - Find your PostgreSQL instance in Bytebase

1. Visit [https://70ca-154-9-204-36.ngrok.io/](https://70ca-154-9-204-36.ngrok.io/) in your browser, and login using your admin account created from the previous article.
![bb-login](/static/blog/database-change-management-with-postgresql-and-github/bb-login.webp)

2. If you have followed the last article, you should have a Project `Sample Project` and a database `demo`.
![bb-home](/static/blog/database-change-management-with-postgresql-and-github/bb-home.webp)

## Step 3 - Connect Bytebase with GitHub.com

1. Click **Settings** on the top bar, and then click **Workspace** > **Version Control**. Choose **GitHub.com** and click **Next**.
![bb-settings-vc-step1](/static/blog/database-change-management-with-postgresql-and-github/bb-settings-vc-step1.webp)

2. Follow the instructions within **STEP 2**, and in this tutorial, we will use a personal account instead of an organization account. The configuration is similar.
![bb-settings-vc-step2](/static/blog/database-change-management-with-postgresql-and-github/bb-settings-vc-step2.webp)

3. Go to your GitHub account. Click **Settings** on the dropdown menu.
![gh-settings-dropdown](/static/blog/database-change-management-with-postgresql-and-github/gh-settings-dropdown.webp)

4. Click **Developer Settings** at the bottom of the left side bar. Click **OAuth Apps**, and click **New OAuth App**.
![gh-oauth-apps](/static/blog/database-change-management-with-postgresql-and-github/gh-oauth-apps.webp)

5. Fill **Application name** and then copy the **Homepage** and **Authorization callback URL** in Bytebase and fill them. Click **Register application**.
![gh-register-oauth](/static/blog/database-change-management-with-postgresql-and-github/gh-register-oauth.webp)

6. After the OAuth application is created successfully. Click **Generate a new client secret**. Copy **Client ID** and this newly generated client secret and paste them back in Bytebase.
![gh-copy-client-id](/static/blog/database-change-management-with-postgresql-and-github/gh-copy-client-id.webp)
![bb-vc-client-id](/static/blog/database-change-management-with-postgresql-and-github/bb-vc-client-id.webp)

7. Click **Next**. You will be redirected to the confirmation page. Click **Confirm and add**, and the Git provider is successfully added.
![gh-auth](/static/blog/database-change-management-with-postgresql-and-github/gh-auth.webp)
![bb-settings-vc-step3](/static/blog/database-change-management-with-postgresql-and-github/bb-settings-vc-step3.webp)

## Step 4 - Enable GitOps workflow with PostgreSQL
1. Go to project `Sample Project`, click **Version Control**, and choose **GitOps Workflow**. Click **Configure GitOps**.
![bb-project-vc-gitops](/static/blog/database-change-management-with-postgresql-and-github/bb-project-vc-gitops.webp)

2. Choose GitHub.com - the provider you just added. It will display all the repositories you can manipulate. Choose `pg-test-bb-local`.
![bb-project-vc-github](/static/blog/database-change-management-with-postgresql-and-github/bb-project-vc-github.webp)
![bb-project-vc-github-local](/static/blog/database-change-management-with-postgresql-and-github/bb-project-vc-github-local.webp)

3. Keep the default setting, and click **Finish**.
![bb-project-vc-gitops-enabled](/static/blog/database-change-management-with-postgresql-and-github/bb-project-vc-gitops-enabled.webp)

## Step 5 - Change schema for PostgreSQL by pushing SQL schema change files to GitHub
1. In your GitHub repository `pg-test-bb-local`, create a folder `bytebase`, then create a subfolder `Prod`, and create an sql file following the pattern `{{ENV_NAME}}/{{DB_NAME}}##{{VERSION}}##{{TYPE}}##{{DESCRIPTION}}.sql`. It is the default configuration for file path template setting under project version control.
   
   `demo##202316410000##ddl##create_t2.sql`
   - `Prod` corresponds to {{ENV_NAME}}
   - `demo` corresponds to {{DB_NAME}}
   - `202316410000` corresponds to {{VERSION}}
   - `ddl` corresponds to {{TYPE}}
   - `create_t2` corresponds to {{DESCRIPTION}}
  
   Paste the sql script in it.

```bash
CREATE TABLE
   "public"."t2" (
   "id" integer NOT NULL,
   "name" character varying(255) NOT NULL,
   PRIMARY KEY ("id")
);

COMMENT
   ON COLUMN "public"."t2"."id" IS 'ID';
```
![vsc-sql](/static/blog/database-change-management-with-postgresql-and-github/vsc-sql.webp)

1. Commit and push this file.
2. Go to Bytebase, and go into project `Sample Project`. You’ll find there is a new `Push Event` and a new `issue 105` created.
![bb-project-push-issue](/static/blog/database-change-management-with-postgresql-and-github/bb-project-push-issue.webp)

1. Click issue/105 and go the issue page, you’ll see
   - The issue is created via GitHub.com
   - The issue is waiting for your approval because it’s on `Prod` environment where manual approval is required by default.
   - The SQL is exactly the one we have committed to the GitHub repository.
   - The Creator is `A`, because the GitHub user you use to commit the change has the same email address found in the Bytebase member list.
![bb-issue-demo](/static/blog/database-change-management-with-postgresql-and-github/bb-issue-demo.webp)

1. Click **Approve**, and the SQL will execute. Click **Resolve issue**, and the issue will be `Done`.
![bb-issue-done](/static/blog/database-change-management-with-postgresql-and-github/bb-issue-done.webp)

1. Click **View change**, you could view the schema diff.
![bb-show-diff](/static/blog/database-change-management-with-postgresql-and-github/bb-show-diff.webp)

1. Go to GitHub repository, you will see besides your committed sql, there is a `.demo##LATEST.sql` file. Because you have configured `Schema path template` before, Bytebase will write back the latest schema to that specified path after completing the schema change. Thus you have access to an update-to-date full schema at any time.
![gh-LATEST](/static/blog/database-change-management-with-postgresql-and-github/gh-LATEST.webp)

## Summary and Next

Now you have tried out GitOps workflow, which will store your PostgreSQL schema in GitHub and trigger the change upon committing the change to the repository, to bring your PostgreSQL change workflow to the next level of Database DevOps - [Database as Code](blog/database-as-code).

In real world scenario, you might have separate features and main branches corresponding to your dev and production environment, you can check out [GitOps with Feature Branch Workflow](/docs/how-to/workflow/gitops-feature-branch) to learn the setup. Have a try and look forward to your feedback!