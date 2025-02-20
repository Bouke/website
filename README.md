# OpenTTD's website

[![GitHub License](https://img.shields.io/github/license/OpenTTD/website)](https://github.com/OpenTTD/website/blob/main/LICENSE)
[![GitHub Tag](https://img.shields.io/github/v/tag/OpenTTD/website?include_prereleases&label=stable)](https://github.com/OpenTTD/website/releases)
[![GitHub commits since latest release](https://img.shields.io/github/commits-since/OpenTTD/website/latest/main)](https://github.com/OpenTTD/website/commits/main)

[![GitHub Workflow Status (Testing)](https://img.shields.io/github/workflow/status/OpenTTD/website/Testing/main?label=main)](https://github.com/OpenTTD/website/actions?query=workflow%3ATesting)
[![GitHub Workflow Status (Publish Image)](https://img.shields.io/github/workflow/status/OpenTTD/website/Publish%20image?label=publish)](https://github.com/OpenTTD/website/actions?query=workflow%3A%22Publish+image%22)
[![GitHub Workflow Status (Deployments)](https://img.shields.io/github/workflow/status/OpenTTD/website/Deployment?label=deployment)](https://github.com/OpenTTD/website/actions?query=workflow%3A%22Deployment%22)

[![GitHub deployments (Staging)](https://img.shields.io/github/deployments/OpenTTD/website/staging?label=staging)](https://github.com/OpenTTD/website/deployments)
[![GitHub deployments (Production)](https://img.shields.io/github/deployments/OpenTTD/website/production?label=production)](https://github.com/OpenTTD/website/deployments)


This is the main website for OpenTTD, as can be seen on [https://www.openttd.org/](https://www.openttd.org/).
`main` is always deployed on [staging](https://www-staging.openttd.org/).

This is a [Jekyll](https://jekyllrb.com/) website, and is served by nginx as a static site.

## Development

### Populating _downloads

By default `_downloads` is empty, so when building the website locally there is no latest stable/nightly.
This can be resolved by running `fetch_downloads`.
This script will download the latest available binaries, and populate `_downloads`.

```bash
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
.venv/bin/python -m fetch_downloads
```

### Running a local server

If you do not want to run a server, but just build the current site, replace `serve` with `build` in the examples below.

Under `_site` Jekyll will put the compiled result in both `serve` and `build`.

- Follow [jekyll installation](https://jekyllrb.com/docs/installation/)
- Run `bundle install`
- Run `JEKYLL_ENV=production jekyll serve`

### Running via Docker

```bash
docker run --rm -v "$(pwd)":/srv/jekyll -it -p 127.0.0.1:4000:4000 -e JEKYLL_ENV=production jekyll/jekyll jekyll serve
```

## Docker image

This repository in the end produces a Docker image which is started in production.
The Dockerfile is a multistage Dockerfile to get to this result.

1) Fetch the downloads.
2) Create the HTML website via Jekyll.
3) Prepare nginx with static files.

The result is a very small image (~50 MiB) with only static HTML sites.
After merging into `main`, [GitHub Actions](https://github.com/OpenTTD/website/actions) automatically publishes a new image on [GitHub Container Registry](https://github.com/OpenTTD/website/pkgs/container/website), and automatically deploys it on [staging](https://www-staging.openttd.org/).

To test locally if the Docker will build, you can use:

```bash
docker build --no-cache --pull -t website:test .
```

## FAQ

### I want to make a new blog post

Create a new file in _posts.
Follow the existing format.
Make a Pull Request, have it approved, and merge.
It will automatically show up on [staging](https://www-staging.openttd.org/).
After tagging, it will move to [production](https://www.openttd.org/).

### I am a developer, and want to be on the website

No problem.
Add yourself to _people, and follow the same as the above 'new blog post' section.

### I get an error about ua-parser.js while building

Please set `JEKYLL_ENV` to `production`.
Without this, the symlink `ua-parser.js` is copied, instead of following it.
With `production` it does the right thing.
To recover, remove `_site/static/js/ua-parser.js` manually.

### What is this download-descriptions.yml

On download pages, you notice that every binary has a human readable description.
`windows-win64.exe` is for most people to cryptic.
`Windows XP / Vista / 7 / 8 / 10 (64bit) (installer)` is much more clear.
This file takes care of that mapping, based on the postfix of the file.

### Why the downloads?

Because this is a static website, but we do want to show in the header what the latest version is, we need to find a solution.
We picked a solution where we fetch some files to know what the latest version is, and create a static version out of it.
This means that every time the latest version changes, the website has to be recreated.
As new versions are rare (once or sometimes twice a day at most), it is very cheaper to do it this way.
It avoids any dynamic component in production.

### Why all the nginx redirects?

We used to have a very dynamic website, with tons of URLs.
Because many people have them bookmarked or made automation around them, we set out to not have any regression during migration.
In result, many URLs are being redirected to their new URL, and we should have not a single regression.

### Screenshots in a git repository?

Yes.
By lack of better, we are doing this.
