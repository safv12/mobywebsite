# Moby website

Website for the moby project.

The Moby website is built using Jekyll and published to Github pages.

## Developing locally

In order to build and test locally, run this command in the docs
directory:

```bash
$ docker run -ti -v "$PWD":/usr/src/app -p "4000:4000" starefossen/github-pages
```
Then browser to localhost:4000 to see the rendered site. The site autorefreshes when you modify files locally.

If you want to test blog posts with a data in the future, use the following command instead.

```bash
$ docker run -ti -v "$PWD":/usr/src/app -p "4000:4000" starefossen/github-pages jekyll serve -d /_site --watch --force_polling -H 0.0.0.0 -P 4000 --future true
```

## GitHub API rate limits

This site uses the GitHub API to dynamically fetch project and users profiles. If you start seeing GitHub rate limit messages in the logs such as `API rate limit exceeded`, you need to create a [GitHub Oauth application](https://github.com/settings/developers) for yourself and set the `GITHUB_CLIENT_SECRET` and `GITHUB_CLIENT_SECRET` environment variables with the values for that application for your container. This bumps up the rate limit from 60 to 5000 per whatever time period they are using.

```bash
$ docker run -ti -v "$PWD":/usr/src/app -p "4000:4000" -e "GITHUB_CLIENT_ID=xxx" -e "GITHUB_CLIENT_SECRET=yyy" starefossen/github-pages
```

## Staging

When you push to this repository on the `theme-start` branch, it triggers an autobuild of the `docker/mobywebsite` private image, which is autodeployed to a password protected staging website at [https://mobyweb.docker.team/](https://mobyweb.docker.team/). This is for Docker design team to test big redesigns without changing production. Docker employees can use their docker.com account login to access it.

## Production

The Moby project website container image is built automatically with a private image, `docker/mobywebsite:latest`. Any push or PR mergin to the master branch triggers an autobuild.

The autobuild uses Docker Hub hooks: a [pre_build](https://github.com/moby/mobywebsite/blob/master/docs/hooks/pre_build) hook script uses the `starefossen/github-pages` image to generate the html from markdown, then the project's [Dockerfile](https://github.com/moby/mobywebsite/blob/master/docs/Dockerfile) is used to build the image, adding that generated content to an Nginx image, with some [Nginx config](https://github.com/moby/mobywebsite/blob/master/docs/nginx/default.conf).

A Docker stack managed in Docker Cloud is autoredeployed everytime this image changes, to [http://web.mobywebsite-prod.8d7e1615.svc.dockerapp.io/](http://web.mobywebsite-prod.8d7e1615.svc.dockerapp.io/).

DNS is managed using Amazon Route53. The backend website is fronted by an Amazon CloudFront distribution that does caching and SSL termination. When a new version of the image is built, a [post_push](https://github.com/moby/mobywebsite/blob/master/docs/hooks/post_push) hook invalidates the cloudfront cache for the distribution.

For design or content issues with this website, please log an issue in this repo. For production issues please ping `website@mobyproject.org`.



