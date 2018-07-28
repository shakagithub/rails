# Rails on Convox Example

This repository contains an example Ruby on Rails 4.2 app configured for local development and deployment to Convox.

The following is a step-by-step walkthrough of how the app was configured and why.

### Dockerfile

#### convox/rails Docker image

The generated Dockerfile inherits from the [convox/rails Docker image](https://hub.docker.com/r/convox/rails/), which has all the packages and configuration necessary to run your Rails app both locally and in production. This includes:

* OS libraries to support PostgreSQL, MySQL, and sqlite3 databases
* nodejs for compiling Javascript assets
* nginx for proxying client connections
* a Convox-friendly nginx config file
* a convox.rb file for logging to STDOUT
* a bin/web script for booting the app

#### how Dockerfile describes the build

Starting from the `convox/rails` image, the [generated Dockerfile](https://github.com/convox-examples/rails/blob/master/Dockerfile) executes the remaining build steps that your Rails app needs. There are basically 3 steps in this process, and they are executed in a particular order to take advantage of Docker's build caching behavior.

1. `Gemfile` and `Gemfile.lock` are copied and `bundle install` is run. This happens first because it is slow and something that's done infrequently. After running once, this step will be cached unless the cache is busted by later edits to `Gemfile` or `Gemfile.lock`.

2. All the files necessary for the Rails asset pipeline are copied, and assets are built. Again, this is done early in the build process to optimize caching. The asset building step will only be run in the future if these files have changed.

3. The rest of the application source is copied over. These files will change frequently, so this step of the build will very rarely be cached.

### convox.yml

The [convox.yml](https://github.com/convox-examples/rails/blob/master/convox.yml) file explains how to run the application. This generated file describes a `web` service which will be your main Rails web process.

## Deploying the application

After [installing a Rack](https://convox.com/docs/installing-a-rack/) create an app and deploy your code to it:

```bash
convox apps create myapp
convox deploy -a myapp
```

Don't forget to run the migrations:

```bash
$ convox run web rake db:migrate -a myapp
```
