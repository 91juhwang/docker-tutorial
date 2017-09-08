# Docker Tutorial

## How to add Docker to an exisiting rails app

1. `echo "FROM rails:onbuild" > Dockerfile`
2. `docker build -t file_name .`
3. `docker run -p 3000:3000 demo` <-- Check if build was successful
4. `$ echo ".git" > .dockerignore` <-- add .git files to .ignore for Docker

It is ready to be delivered and deployed as `Docker image`. Images can be used to launch application instances in development environments, thus reducing the need for a local Ruby/Bundler setup. A continuous delivery pipeline could use the Dockerfile to automate builds of the image.

The `rails:onbuild` command provides many things by default.

  * System dependencies and recommended packages are already preinstalled.
  * Rails server is launched when container is started.
  * Updates to the Gemfile and Gemfile.lock are recognized during builds, and the appropriate bundle install step is triggered when needed.
  * Application code is installed into /usr/src/app on a virtual filesystem. If the Gemfile hasn’t been changed since the previous build, the previous, cached bundle install step is used.

## Little More Advanced Configuration

The above build is good for simple & getting started with dockerization. However there are limitations.

### Limitation for above implementation

  * A recent, stable version of Ruby is used by default, which may not always be acceptable for your application.
  * Gemfile.lock should be updated outside the build process. This fairly standard Bundler practice becomes redundant for properly organized Docker-based developments.
  * Support for installing extra system-level packages is limited. While it is technically possible to add packages with additional RUN apt-get… lines to the Dockerfile, these packages will not be available during the bundle install step due to the ordering of the Dockerfile lines. Adding packages to this Dockerfile would also increase the time for each subsequent build, since Docker would not be able to cache those commands.

TLDR; -- If the image should be built in new OS, totally different or new environment, above implementation does not work or the user have to manually install Ruby & core system dependencies.

### Configuration from pre-generaged rails:onbuild Dockerfile

Refer to below Dockerfile script

```ruby
FROM ruby:2.3

# throw errors if Gemfile has been modified since Gemfile.lock
# RUN bundle config --global frozen 1

RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

EXPOSE 3000
CMD ["rails", "server", "-b", "0.0.0.0"]

RUN apt-get update && apt-get install -y nodejs --no-install-recommends && rm -rf /var/lib/apt/lists/*
RUN apt-get update && apt-get install -y mysql-client postgresql-client sqlite3 --no-install-recommends && rm -rf /var/lib/apt/lists/*

COPY Gemfile /usr/src/app/

# Uncomment the line below if Gemfile.lock is maintained outside of build process
# COPY Gemfile.lock /usr/src/app/


RUN bundle install

COPY . /usr/src/app
```

This Dockerfile does basically the same thing as above 1 liner, but will run the `bundle install` inside the container, which will allow us to not manually run bundle install everytime and check consistency in Gemfile.lock. Because we will run `bundle` ourself, we do not need Gemfile.lock.

So add it to `.dockerignore` & `.ignore`

Rebuild the image to ensure it works.

`docker build -t demo .`

### Push

* `docker login`

* check images by `docker images`

* tag it by `docker tag username/image_name`

* push by `docker push repository`

Image to above code is below
[DockerHub](https://hub.docker.com/u/91juhwang/)

## Terminology

### Registry vs. Index

* Index: manages user accounts, permissions, search, tagging, just like Github

* Registry: stores and serves up the actual image assets and delegates authentication to index.

When `docker search` is ran, it is searching through `index` not the `registry`.

When `docker push or pull` is ran, the `index` determines if you are allowed to do theses actions, and `registry` is the piece that stores or sends it after `index` has approved.
