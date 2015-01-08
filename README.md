"Let's dockerize all that shit!" -- Unknown author (probably me)

# The reason

Let's assume WSGI application written in Python (like Django, for example). It has some dependencies installed via `pip`.

When new release is build, the usual deployment procedure is following:

1. Pull new code
2. Install dependencies
3. Install new version of app
4. Post-install actions (like migrate database, compile messages, ...)
5. Reload WSGI

## Problem 1 -- Revert

This procedure easily works when moving forward. However, there's no quick way to move **backward** when something breaks.

The installation of dependencies could take significant time or worse: the old versions of packages might not be available anymore. They could be kept localy as a workaround, but still: When something doesn't work, the speed of reverting to last working state matters.
 

## Problem 2 -- Environments

"It works for me" is a joke used during development. Don't use it in production.

The difference between development, staging and production environment should be as minimal as possible. However, even when you change only the necessary settings, there are still some caveats, like developing on different platform (hello OSX users with case-insensitive filesystems deploying to case-sensitive ones).

Still, what about using the **same** release image for testing in staging and deployment to production.

# The test -- Docker

Let's use docker to:

1. Create development environment
2. Releasing images which are deployed to staging, tested and when everything is fine, the same image is deployed to production.
3. Keeping old containers for quick revert of changes

## Why docker?

- Everything (dependencies, application, wsgi app) can be kept inside docker, exposing only one port with http.
- Lightweight containers
- It's new, so it must be good (no, really, it's a new technology, let's find out how it works and how take advantage of it)

## Solution

The tested stack is composed from nginx and uwsgi as follows:

1. nginx
2. uwsgi fastrouter with subscription server
3. pool of uwsgi workers running in docker containers

The fastrouter feature is used to achieve "graceful reloads", switching running containers without loosing (refusing) any requests from clients. (**TODO**: Is there any other solution?)