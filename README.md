<div align="center">
  <h1>Medusa Platform</h1>
</div>

<div align="center">
  <p>Launch all Medusa services on the cloud server.</p>
</div>

## About

### What is Medusa Platform?

The Medusa platform aims to be the easiest way to deploy essential Medusa services on your cloud server for production-ready:
- [Medusa API (Comes with plugins such as Inventory, Stripe, SendGrid, etc.)](https://github.com/chrislaai/medusa-api-template/tree/prod)
- [Medusa Admin Dashboard (Under development mode)](https://github.com/chrislaai/medusa-admin-template/tree/prod)
- [Medusa Storefront (Performant storefront with Next.js 14)](https://github.com/chrislaai/medusa-storefront-template/tree/prod)
- Nginx (HTTP and reverse proxy server)
- MinIO (Object Storage)
- Meilisearch (Search Engine)
- The necessary databases, cache, etc.

Currently I would like to use [Submodules](https://git-scm.com/book/en/v2/Git-Tools-Submodules) to keep the project more flexible. If you want to use it in production, you should create your own Private repo, delete the existing Submodules and then use Submodules to add your own Private repo to the Platform repo to use them.

<details><summary>Not familiar with removing and replacing Submodules?</summary>
<p>
  
This assumes that you already have a Git repository containing a submodule and want to replace the submodule with a new one.

1. Delete existing submodules:
```shell
git submodule deinit -f <submodule-path>  # git submodule deinit -f apps/api
git rm --cached <submodule-path> # git rm --cached apps/api
rm -rf .git/modules/<submodule-path> # rm -rf .git/modules/apps/api
```

2. Add new submodule:
```shell
git submodule add <new-submodule-url> <submodule-path> # git submodule add https://github.com/chrislaai/medusa-api-template.git apps/api

```

*Or you want to specify a branch using the `-b` parameter:*

```shell
git submodule add -b <branch-name> <new-submodule-url> <submodule-path> # git submodule add -b prod https://github.com/chrislaai/medusa-api-template.git apps/api
```

</p>
</details>

## Prerequisites
- Cloud Server
  - In my case I used [Hostinger](https://www.hostinger.com/vps-hosting?REFERRALCODE=1CHIU76) VPS Hosting KVM 1 Plan and it worked fine
- [Docker](https://docs.docker.com/engine/install/)
- [Node.js (Installing Node.js via package manager)](https://github.com/nvm-sh/nvm)
- [PM2](https://pm2.keymetrics.io/)
- [Origin CA certificates](https://developers.cloudflare.com/ssl/origin-configuration/origin-ca/)

*Note that `Node.js` and `PM2` only required if you wish to deploy Medusa `apps` without using docker.*

## How to run it?

There are two deployment options for this repository, since we have placed `apps` and `common-services` in different folders, you can easily choose to deploy everything using Docker, or deploy `common-services` using Docker and then use Node.js + PM2 deploys Medusa `apps`.

## Getting started

```bash
git clone https://github.com/chrislaai/medusa-platform-template.git --recursive --jobs 2
```

*Place your Origin CA certificate files in the [certs](/common-services/certs) folder and update each certificate file name and `server_name` in the [common-services.conf](/common-services/nginx/common-services.conf) file.*

```bash
cd ~/medusa-platform-template/common-services
```

```bash
cp .env.template .env
```

*To run Meilisearch in production mode, you’ll need to configure an API key for secure use. You can create your own `MEILI_MASTER_KEY` (at least 16 characters), or generate one using the following openssl command `openssl rand -hex 32`.*

```bash
vim .env
```

```bash
docker network create medusa
```

```bash
docker compose up -d
```

## Configuring MinIO storage

1. Access https://minio-console.example.com
2. Create a new bucket and call it as you did in `MINIO_BUCKET` variable.
3. Set its `Access Policy` settings to `public`.
4. Create a new `Access Keys` pair and copy them to `MINIO_ACCESS_KEY` and `MINIO_SECRET_KEY` variables.

## Run the Medusa apps with Docker

*Update each certificate file name, `server_name` and `proxy_pass` parameter in the [apps.conf](/common-services/nginx/apps.conf) file.*

```bash
cd ~/medusa-platform-template/apps
```

```bash
cp .env.template .env
```

```bash
vim .env
```

*You need to update the image hostname to your own by editing file [next.config.js](apps/storefront/next.config.js)*

```bash
docker compose up -d
```

## Run the Medusa apps with Node.js and PM2

*Update each certificate file name,`server_name` and `proxy_pass` parameter in the [apps.conf](/common-services/nginx/apps.conf) file.*

<details><summary>Make sure you have installed PM2, Yarn and @medusajs/medusa-cli in "global" mode</summary>
<p>

```shell
npm install -g yarn pm2 @medusajs/medusa-cli
```

</p>
</details>

```bash
cd ~/medusa-platform-template/apps/api
```

```bash
cp .env.template .env
```

```bash
vim .env
```

```bash
yarn
```

<details><summary>You can always run migrations before yarn start</summary>
<p>

```shell
npx medusa migrations run
```

</p>
</details>

*You can choose to use the command `npx medusa seed -f ./data/seed.json` add demo data to your Medusa backend or at least create an admin account to log in with the command `npx medusa user -e admin@medusa-test.com -p supersecret`.*

```bash
pm2 start yarn --name "api" -- start
```

```bash
cd ~/medusa-platform-template/apps/admin
```

```bash
cp .env.template .env
```

```bash
vim .env
```

```bash
yarn
```

```bash
pm2 start yarn --name "admin" -- dev
```

```bash
cd ~/medusa-platform-template/apps/storefront
```

```bash
cp .env.template .env
```

```bash
vim .env
```

```bash
yarn
```

*You need to update the image hostname to your own by editing file [next.config.js](apps/storefront/next.config.js)*

```bash
yarn build
```

```bash
pm2 start yarn --name "storefront" -- start
```

```bash
pm2 save
```
