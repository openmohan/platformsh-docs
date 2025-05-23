---
title: Deploy Next.js on {{% vendor/name %}}
sidebarTitle: Next.js
weight: -97
layout: single
description: |
    Complete the last required steps to successfully deploy Next.js on {{% vendor/name %}}.
---

{{< note title="Note" theme="info" >}}

Before you start, check out the [{{% vendor/name %}} demo app](https://console.upsun.com/projects/create-project) and the main [Getting started guide](/get-started/here/_index.md).
They provide all of the core concepts and common commands you need to know before using the materials below.

{{< /note >}}

{{% guides/requirements name="Next.js" %}}

## 1. Create a Next.js app

To create your Next.js app, follow these steps.

1. Follow the Next.js [installation guide](https://nextjs.org/docs/getting-started/installation).
   To fast track the process, run the following commands:

   ```bash {location="Terminal"}
   npx create-next-app@latest myapp
   ```

2. To initialize the local Git repository and commit local files, run the following commands:

   ```bash {location="Terminal"}
   cd myapp
   git init
   git add .
   git commit -m "Init Next.js application."
   ```

{{< note theme="info" >}}
You can view the running app locally by running `npm run dev`.
{{< /note >}}

## 2. Create a new project

To create a project on {{% vendor/name %}},
follow these steps.

{{< note title="Remember" >}}
After creating your {{% vendor/name %}} project, copy your new **project ID** for later use.
{{< /note >}}

{{< codetabs >}}
+++
title=Using the CLI
+++
To create a new project with the {{% vendor/name %}} CLI, use the following command and follow the prompts:

```bash {location="Terminal"}
{{% vendor/cli %}} project:create
```

{{< note >}}

When creating a new project using the {{% vendor/name %}} CLI command `project:create`,
you are asked if you want to set the local remote to your new project. Enter **Yes (y)**.

Your local source code is automatically linked to your newly created {{% vendor/name %}} project
through the creation of a `.{{% vendor/cli %}}/local/project.yaml`.
This file contains the corresponding `<projectId>` for the {{% vendor/name %}} CLI to use,
and sets a Git remote to `{{% vendor/cli %}}`.

{{< /note >}}

<--->
+++
title=Using the Console
+++

1. Create an organization or select an existing one.

2. Click **Create from scratch**.

3. Fill in details like the project name and [region](/development/regions.md).

   {{% note %}}

   You can define resources for your project later on, after your first push.

   {{% /note %}}

4. To link your local source code to your new {{% vendor/name %}} project,
   run the following command:

   ```bash {location="Terminal"}
   {{% vendor/cli %}} project:set-remote <projectId>
   ```

   This command adds a new remote called `{{% vendor/cli %}}` to your local Git repository,
   which is equivalent to the following commands:

   ```bash {location="Terminal"}
   git remote
   origin
   {{% vendor/cli %}}
   ```

   It also creates a new `.{{% vendor/cli %}}/local/project.yaml` file that contains the `<projectId>`
   for the `{{% vendor/cli %}}` CLI to use.

   {{< note theme="info" title="Tip" >}}

   If you forget your `<projectId>`, run the following command and find your project in the list:

   ```bash {location="Terminal"}
   {{% vendor/cli %}} project:list
   ```
   {{< /note >}}

{{< /codetabs >}}

## 3. Choose your Git workflow

You can use {{% vendor/name %}} projects as a classic Git repository,
where you are able to push your source code in different ways,
using either the Git CLI or the {{% vendor/name %}} CLI.
You can choose which way —or Git workflow— you want to use for your project from the following options:

- Your project source code is **hosted on a {{% vendor/name %}} Git repository**
- Your project source code is **hosted on your own GitHub repository**

{{< codetabs >}}
+++
title={{% vendor/name %}} Git repository
+++
For the rest of this guide, you will use the normal Git workflow (`git add . && git commit -m "message" && git push {{% vendor/cli %}}`) to commit your source code changes to Git history.
You will also use the {{% vendor/name %}} CLI to deploy your [{{% vendor/name %}} environment](/environments.html) with the latest code updates.

<--->
+++
title=GitHub repository
+++
{{% vendor/name %}} provides a [Github integration](integrations/source/github.md) that allows your {{% vendor/name %}} project to be fully integrated with your Github repository.
This enables you, as a developer, to use a normal Git workflow (`git add . && git commit -m "message" && git push`) to deploy your environment—with no need to connect to the {{% vendor/name %}} Console.

{{< note >}}
Make sure you complete the following steps before adding a [Github integration](integrations/source/github.md):

1. Create a Git repository in your own organization following the relevant
   [Github repository creation guide](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-new-repository).

2. Create a [Github integration](integrations/source/github.md).

3. Add a Git remote to your local project, from the root of your Next.js directory.</br>
   To do so, run the following commands:

    ```bash {location="Terminal"}
    git remote add origin <urlOfYourOwnGitHubRepo>
    git add . && git commit -m "init next.js"
    git push origin
    ```
{{< /note >}}

{{< /codetabs >}}

## 4. Configure your project

To host your Next.js application on {{% vendor/name %}},
you need to have a few YAML configuration files at the root of your project.
These files manage your app's behavior.
They are located in a `.{{% vendor/cli %}}/` folder at the root of your source code
and structured in a similar way to this:

```txt
myapp
├── .{{% vendor/cli %}}
│   └── config.yaml
├── [.environment]
└── <project sources>
```

To generate these files, run the following command at the root of your project:

``` {location="Terminal"}
{{% vendor/cli %}} project:init
```

Follow the prompts, and you should result in such a config file.

```yaml {configFile="apps"}
applications:
  myapp:
    source:
      root: "/"
    type: "nodejs:{{% latest "nodejs" %}}"
    mounts:
      "/.npm":
        source: "storage"
        source_path: "npm"
    hooks:
      build: |
        set -eux
        npm i
        npm run build
    web:
      commands:
        start: "npx next start -p $PORT"
      upstream:
        socket_family: tcp
      locations:
        "/":
          passthru: true
routes:
  "https://{default}/": { type: upstream, upstream: "myapp:http" }
  "http://{default}/": { type: redirect, to: "https://{default}/" }
```

As an example, above is the minimum configuration needed to deploy a Next.js application on {{% vendor/name %}} without any services.
Depending on your answers to the prompts, you may also have `relationships` and `services` defined.

To commit your new files, run the following commands:

```bash {location="Terminal"}
git add .
git commit -m "Add {{% vendor/name %}} config files"
```

## 5. Deploy

And just like that, it’s time to deploy!

Depending on the Git workflow you chose at the beginning of this tutorial,
there are two ways to deploy your source code changes.

{{< codetabs >}}

+++
title=Using {{% vendor/name %}} Git repository
+++

You can push your code using the normal Git workflow (`git add . && git commit -m "message" && git push`).
This pushes your source code changes to your `{{% vendor/cli %}}` remote repository.
Alternatively, you can use the following {{% vendor/name %}} CLI command:

```bash {location="Terminal"}
{{% vendor/cli %}} push
```

<--->
+++
title=Using third-party Git repository
+++

When you choose to use a third-party Git hosting service, the {{< vendor/name >}} Git
repository becomes a read-only mirror of the third-party repository. All your
changes take place in the third-party repository.

Add an integration to your existing third-party repository:

- [BitBucket](/integrations/source/bitbucket.md)
- [GitHub](/integrations/source/github.md)
- [GitLab](/integrations/source/gitlab.md)

If you are using an integration, on each code updates,
use the normal Git workflow (`git add . && git commit -m "message" && git push`) to push your code to your external repository.
To do so, run the following command:

```bash {location="Terminal"}
git push origin
```

Your GitHub, GitLab, or Bibucket integration process then automatically deploys changes to your environment.
If you're pushing a new Git branch, a new environment is created.

{{< /codetabs >}}

{{% vendor/name %}} then reads your configuration files,
and deploys your project using [default container resources](/manage-resources/resource-init.md).
If you don't want to use those default resources,
define your own [resource initialization strategy](/manage-resources/resource-init.md#specify-a-resource-initialization-strategy),
or [amend those default container resources](/manage-resources/adjust-resources.md) after your project is deployed.

Et voilà, your Next.js application is live!

{{< note title="Tip" theme="info" >}}

Each environment has its own domain name.
To open the URL of your new environment, run the following command:

   ```bash {location="Terminal"}
   {{% vendor/cli %}} environment:url --primary
   ```
{{< /note >}}

## 6. Make changes to your project

Now that your project is deployed, you can start making changes to it.
For example, you might want to fix a bug or add a new feature.

In your project, the `main` branch always represents the production environment.
Other branches are for developing new features, fixing bugs, or updating the infrastructure.

To make changes to your project, follow these steps:

1. Create a new environment (a Git branch) to make changes without impacting production:

   ```bash {location="Terminal"}
   {{% vendor/cli %}} branch feat-a
   ```

   This command creates a new local `feat-a` Git branch based on the main Git branch,
   and activates a related environment on {{< vendor/name >}}.
   The new environment inherits the data (service data and assets) of its parent environment (the production environment here).

2. Make changes to your project.
   For example, edit the `views/index.jade` file and make the following changes:

   ```diff
   diff --git a/views/index.jade b/views/index.jade
   index 3d63b9a..77aee43 100644
   --- a/views/index.jade
   +++ b/views/index.jade
   @@ -2,4 +2,4 @@ extends layout

    block content
      h1= title
   -  p Welcome to #{title}
   +  p Welcome to #{title} on {{% vendor/name %}}
   ``

3. Commit your changes:

   ```bash {location="Terminal"}
   git add views/index.jade
   git commit -m "Update index page view."
   ```

4. Deploy your changes to the `feat-a` environment:

   ```bash {location="Terminal"}
   {{% vendor/cli %}} push
   ```

5. Iterate by changing the code, committing, and deploying.
   When satisfied with your changes, merge them to the main branch,
   and remove the feature branch:

   ```bash {location="Terminal"}
   {{% vendor/cli %}} merge
       Are you sure you want to merge feat-a into its parent, main? [Y/n] y
   {{% vendor/cli %}} checkout main
   git pull {{% vendor/cli %}} main
   {{% vendor/cli %}} environment:delete feat-a
   git fetch --prune
   ```

   Note that deploying to production is fast because the image built for the `feat-a` environment is reused.

   For a long running branch, to keep the code up-to-date with the main branch, use `git merge main` or `git rebase main`.
   You can also keep the data in sync with the production environment by using `{{% vendor/cli %}} env:sync`.

## Further resources

### Documentation

- [JavaScript/Node.js documentation](/languages/nodejs/_index.md)
- [Managing dependencies](/languages/nodejs/_index.md#dependencies)

### Community content

- [Next.js topics](https://support.platform.sh/hc/en-us/search?utf8=%E2%9C%93&query=nextjs)
- [Node.js topics](https://support.platform.sh/hc/en-us/search?utf8=%E2%9C%93&query=node)
- [JavaScript topics](https://support.platform.sh/hc/en-us/search?utf8=%E2%9C%93&query=js)

### Blogs

- [A quick-start guide on hosting Next.js on {{% vendor/name %}}](https://upsun.com/blog/setting-up-next-js-on-upsun/)
