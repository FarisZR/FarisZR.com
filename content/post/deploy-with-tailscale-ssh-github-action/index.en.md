---
title: Deploy Projects Securely Using Tailscale SSH and GitHub Actions
categories: 
    - quick-snippets
tags:
    - github
    - github actions
    - tailscale
    - deployment
date: 2024-03-31
slug: deploy-with-tailscale-ssh-github-action
aliases:
    - /en/deploy-with-tailscale-ssh-github-action/
summary: deploy to your server without exposing an ssh port using tailscale-ssh-deploy, a GitHub action that utilizes Tailscale SSH.
image: thumbnail.jpg
keywords: 
    - tailscale
    - github 
    - ssh
---

If you want to deploy your project to a server without exposing it or manually managing ssh keys, you can use Tailscale SSH!

Here is a quick guide to using my Github action [tailscale-ssh-deploy](https://github.com/FarisZR/tailscale-ssh-deploy) to deploy files to a server using Tailscale SSH (not normal ssh).

### Deploying a static site

For example, we will deploy a Hugo static site using this action

```yaml
on:
  push:
jobs:
  build-and-deploy:
    name: build and deploy
    runs-on: ubuntu-latest
    steps:
      - name: checkout
        uses: actions/checkout@v3
        with:
          fetch-depth: 0

      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          extended: true

      - name: Build
        run: hugo --minify -v --gc --destination generated

        - name: Setup Tailscale
        uses: tailscale/github-action@v2
        with:
            hostname: Github-actions
            oauth-client-id: ${{ secrets.TS_OAUTH_CLIENT_ID }}
            oauth-secret: ${{ secrets.TS_OAUTH_SECRET }}
            tags: tag:ci

        - name: Start Deployment
        uses: FarisZR/tailscale-ssh-deploy@v1
        with:
            remote_host: USER@LOCAL-IPV4-ADDRESS-TAILSCALE
            directory: generated
            remote_destination: /var/www/html
            post_upload_command: systemctl restart nginx
```

The relevant part for us are the last two actions.
First, we set up Tailscale using an Oauth client ID and secret that you generate in the settings page of your Tailnet [more details](https://tailscale.com/kb/1215/oauth-clients).

Now make sure you specify the user and use the Tailscale IP address of the server, as I didn't find using MagicDNS hostnames to be very reliable.

Then you just define the directory to upload to in `directory` and where it should be uploaded into `remote_destination`

You can use `post_upload_command` to specify a command to run after a successful upload, as an example I added a command to restart nginx.

That should do it! Now you have a workflow that automatically deploys your project to a server without exposing an ssh service or managing ssh keys!