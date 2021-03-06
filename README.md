# ASP.NET Core / React SPA Template App

This app is a template application using ASP.NET Core for a REST/JSON API server and React for a web client.

## Overview of Stack
- Server
  - ASP.NET Core
  - PostgresSQL
  - Entity Framework Core w/ EF Migrations
  - JSON Web Token (JWT) authorization with OpenIddict
  - Docker used for development PostgresSQL database
- Client
  - React
  - Webpack for asset bundling and HMR
  - CSS Modules
- Testing
  - xUnit for .NET Core
  - Enzyme for React
- DevOps
  - Ansible playbook for provisioning (Nginx reverse proxy, SSL via Let's Encrypt, PostgresSQL backups to S3)
  - Ansible playbook for deployment

## Setup

1. Install the following:
   - [.NET Core](https://www.microsoft.com/net/core)
   - [Node.js >= 4.0](https://nodejs.org/en/download/)
   - [Ansible >= 2.0](http://docs.ansible.com/ansible/intro_installation.html)
   - [Docker](https://docs.docker.com/engine/installation/)
2. Run `npm install && npm start`
3. Open browser and navigate to [http://localhost:5000](http://localhost:5000).

## Scripts

### `npm install`

When first cloning the repo or adding new dependencies, run this command.  This will:

- Install Node dependencies from package.json
- Install .NET Core dependencies from api/project.json and api.test/project.json (dotnet restore)

### `npm start`

To start the app for development, run this command.  This will:

- Run `docker-compose up` to ensure the Postgres Docker image is up and running
- Run dotnet watch run which will build the app (if changed), watch for changes and start the web server on http://localhost:5000
- Run Webpack dev middleware with HMR via [ASP.NET JavaScriptServices](https://github.com/aspnet/JavaScriptServices)

### `npm test`

This will run the xUnit tests in api.test/ and the Mocha tests in client-react.test/.

### `npm run provision:prod`

 _Before running this script, you need to create a ops/hosts file first.  See the [ops README](ops/) for instructions._

 This will run the ops/provision.yml Ansible playbook and provision hosts in ops/hosts inventory file.  This prepares the hosts to recieve deployments by doing the following:
  - Install Nginx
  - Generate a SSL certificate from [Let's Encrypt](https://letsencrypt.org/) and configure Nginx to use it
  - Install .Net Core
  - Install Supervisor (will run/manage the ASP.NET app)
  - Install PostgreSQL
  - Setup a cron job to automatically backup the PostgresSQL database, compress it, and upload it to S3.
  - Setup UFW (firewall) to lock everything down except inbound SSH and web traffic
  - Create a deploy user, directory for deployments and configure Nginx to serve from this directory

### `npm run deploy:prod`

_Before running this script, you need to create a ops/hosts file first.  See the [ops README](ops/) for instructions._

This script will:
 - Build release Webpack bundles
 - Package the .NET Core application in Release mode (dotnet publish)
 - Run the ops/deploy.yml Ansible playbook to deploy this app to hosts in /ops/hosts inventory file.  This does the following:
  - Copies the build assets to the remote host(s)
  - Updates the `appsettings.release.json` file with PostgresSQL credentials specified in ops/hosts file and the app URL (needed for JWT tokens)
  - Restarts the app so that changes will be picked up

## Credit

The following posts were helpful in setting up this template.

- [Sample for implementing Authentication with a React Flux app and JWTs](https://github.com/auth0-blog/react-flux-jwt-authentication-sample)
- [Angular 2, React, and Knockout apps on ASP.NET Core](http://blog.stevensanderson.com/2016/05/02/angular2-react-knockout-apps-on-aspnet-core/)
- [Setting up ASP.NET v5 (vNext) to use JWT tokens (using OpenIddict)](http://capesean.co.za/blog/asp-net-5-jwt-tokens/)
- [Cross-platform Single Page Applications with ASP.NET Core 1.0, Angular 2 & TypeScript](https://chsakell.com/2016/01/01/cross-platform-single-page-applications-with-asp-net-5-angular-2-typescript/)
- [Stack Overflow - Token Based Authentication in ASP.NET Core](http://stackoverflow.com/questions/30546542/token-based-authentication-in-asp-net-core-refreshed)
- [SPA example of a token based authentication implementation using the Aurelia front end framework and ASP.NET core]( https://github.com/alexandre-spieser/AureliaAspNetCoreAuth)
- [A Real-World React.js Setup for ASP.NET Core and MVC5](https://www.simple-talk.com/dotnet/asp-net/a-real-world-react-js-setup-for-asp-net-core-and-mvc)
