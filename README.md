# Blog

The repo for my personal blog at https://www.RobertJacobson.dev/. I switched from Jekyll to Hugo in February 2024.

Required initial setup:

```shell
brew install hugo
```

One-time [setup is also required for GitHub repo](https://gohugo.io/hosting-and-deployment/hosting-on-github/).

### To build and serve locally:

```shell
hugo server --buildDrafts --disableFastRender
```

### To create a new page:

```shell
hugo new content posts/a-new-post.md
```

### To build for deployment:

```shell
hugo
```

### To deploy:

Pushing to GitHub triggers a GitHub Action that deploys changes automatically.

```shell
git push
```

### Gotcha: [CNAME file](https://docs.github.com/en/github/working-with-github-pages/troubleshooting-custom-domains-and-github-pages#cname-errors)

> Custom domains are stored in a CNAME file in the root of your publishing source.

GitHub creates the CNAME file when you set the custom domain name in the repo settings. Or keep the file in the root of the repo locally.



## References

* [Host on GitHub Pages](https://gohugo.io/hosting-and-deployment/hosting-on-github/)
* [Hugo Quick Reference](https://gohugo.io/quick-reference/)
