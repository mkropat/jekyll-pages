# Jekyll Pages

*Self-host your [GitHub Pages][github-pages] website.*

Jekyll Pages is a set of scripts that will fetch and build your GitHub repos into a website. Once built, you are free to publish the generated site wherever you want. [It is not 100% there yet](#known-limitations), however the goal is to be a drop-in replacement for GitHub Pages.

The scripts are:

- [`get-repos`](get-repos) – looks up all your repos using the API and clones/fetches the latest version
- [`build-site`](build-site) – given a list of repos (piped from `get-repos`), run each [Jekyll][jekyll] build in a [Docker][docker] container
- [`publish`](publish) – run the previous two scripts then `rsync` the resulting website to your server

## Getting Started

### Pre-Requisites

- [Docker][docker-install]
- `sudo apt install curl git jq rsync` / `sudo dnf install curl git jq rsync` / etc.

### Configuration

You need to create a GitHub API token:
 
![new personal access token][api-token-screenshot]

1. Under __Settings__ → __Developer settings__ → __Personal access tokens__, click __Generate new token__.
1. Enter a token description of your choosing.
1. Under the "repo" scope, enable __public_repo__.
1. Click __Generate token__

Once you have the API token, you need to save both your GitHub username and the token to a file named `~/.github-creds` so Jekyll Pages can find it.

Here is one way you could do that:

```
$ (umask 077; cat > ~/.github-creds)
your_username:e9437896d2ded04ce3cf8bbca4757811a3fc5d33
^D
```

(The `umask 077` ensures that the file is readable only by your user.)

### Running

You can build your website and deploy it by running:

```
./jekyll-pages/publish user@yourhost.example.com:/path/to/www
```

Or if you host your website on the same server:

```
./jekyll-pages/publish /var/www/html
```

The deployment is performed by rsync(1). See the [rsync documentation][man-rsync] for the full list of supported destinations.


For convenience, you can create a `PAGES_DEPLOY_DEST` environment variable specifying the default deployment destination. For example:

```
export PAGES_DEPLOY_DEST="user@yourhost.example.com:/path/to/www"
echo 'export PAGES_DEPLOY_DEST="user@yourhost.example.com:/path/to/www"' >> ~/.bashrc
```

Then all you would have to run is:

```
./jekyll-pages/publish
```

### Automated Deployments

You can schedule the publish script to run periodically. Start by editing the crontab:

```
crontab -e
```

Then to run the publish script every night you could add the following entry:

```
0 0 * * * /path/to/jekyll-pages/publish user@yourhost.example.com:/path/to/www
```

## Why self host?

- Full access to all analytics
- Not limited to the [officially blessed plugins/gems][pages-gem]
- Can run any other software you want
- Use any domain you want
- Multi-provider redundancy

Finally, even if you don't use Jekyll Pages, it might give you peace of mind to continue using GitHub Pages, knowing that you always have the option to self-host any time you choose.

## Supported Features

- User and project sites
- [`.nojekyll`][nojekyll] sites
- Repos with submodules

## Known Limitations

I created Jekyll Pages to move my personal site off of GitHub Pages. You should probably consider it alpha quality software. What follows is a list of known limitations.

#### No automated deployment on push

GitHub pages will automatically rebuild your site when you push changes to any corresponding repo. Jekyll Pages must be run manually or on a fixed schedule.

It probably wouldn't be too hard for Jekyll Pages to create a webhook that gets notified whenever a push happens.

#### Breaks if you have multiple *.github.io repos

Jekyll Pages assumes that if a repo is named *.github.io that it is your main website. This is probably a good assumption up until the point you fork someone else's *.github.io website, perhaps to make a PR. At which point, Jekyll Pages will have one of the sites clobber the other in the built website.

It probably wouldn't be too hard to detect when the user has more than one *.github.io repo and require the user to pass an option specifying which one they want to use for their main website.

#### Project sites must use the gh-pages branch

GitHub Pages supports building project sites from different sources:

![github pages options][gh-pages-screenshot]

Jekyll Pages only supports the gh-pages branch source.

I haven't checked to see if this project site configuration is available from the API. If it is, it probably wouldn't be too hard to support the other sources.

#### Hidden files are not published

As a blanket security measure, Jekyll Pages will not copy hidden files (filenames that starts with a `.`) to the output site.

There are probably legitimate use cases for serving hidden files. However, until I am made aware of what those use cases are, I am not going to try to build any support into Jekyll Pages.

#### Does not support organizations / Enterprise GitHub

GitHub Pages can build websites for organizations in addition to user pages. Your organization may also have a self-hosted version of GitHub (Enterprise GitHub), in which case the API would be at a different URL.

It probably wouldn't be hard to support either of these use cases.

## Contributing

[Pull requests](https://github.com/mkropat/jekyll-pages/pulls) are always welcome. [Issues](https://github.com/mkropat/jekyll-pages/issues) too.

As a maintainer I don't have the best track record with hand-holding PRs that need work. That is where you can help. If you see an open PR or issue that could use feedback, please speak up. If you see a PR that looks good but hasn't been tested, please try it out and post a comment whether the change worked for you.

[api-token-screenshot]: https://i.imgur.com/l1dATBs.png
[docker]: https://www.docker.com/
[docker-install]: https://docs.docker.com/install/#supported-platforms
[gh-pages-screenshot]: https://i.imgur.com/H4r65jy.png
[github-pages]: https://pages.github.com/
[jekyll]: https://jekyllrb.com/
[man-rsync]: https://download.samba.org/pub/rsync/rsync.html
[nojekyll]: https://blog.github.com/2009-12-29-bypassing-jekyll-on-github-pages/
[pages-gem]: https://github.com/github/pages-gem
[supported-custom-domains]: https://help.github.com/articles/about-supported-custom-domains/
