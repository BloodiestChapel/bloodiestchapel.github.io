---
title: "Hugo Website Deployment w/ Github Pages"
date: 2026-06-14
draft: false
tags: ["homelab", "website", "github", "pages", "hugo"]
cover:
    image: "/images/banner_hugo-github.jpg"
    alt: "Banner"
    relative: false
---
# Setting Up a Static Site with Hugo and GitHub Pages
## Why I Have My Own Site
There are a few reasons I decided to make my own site.

- As I stated in one of my [posts]({{< ref "/posts/Homelab_Update_6-7-2026.md" >}}), this started more as a means of organizing my homelab services than anything, but I figured since I have the domain and the cloudflare account, I might as well go the extra mile and start up my own website. 

- People ask for my knowledge on some of these subject  a lot, so I might as well also have a centralized spot besides my code on Github for everything I've learned. Another plus, is it gets me used to writing documentation on how I've done what I've done. That way if I need to improve or change something, I can reference this to make sure I'm doing things right and to make sure I improve in the future.

## Prerequisites
There are a few things that are needed before anything can start:
- A cloudflare account, free tier works just fine
- Your own domain, I used porkbun to get my domain for cheap
- A Github account


## Why Hugo/Cloudflare/Github Pages?
- I wanted to use Hugo because after learning so much about HTML, CSS, JavaScript, etc, I came to the realization that it's a bit annoying to write code for a website. While it can be fun, I really just wanted something that's easy and that worked. Christian Lempa actually has a [tutorial](https://www.youtube.com/watch?v=MX4yy1dTVYg&t=896s) where he goes over using Hugo with the blowfish theme. I went with something much easier to use out of the box, called PaperMod, but it's all up to you which theme you want, which is another thing I liked; the customizability. I will likely try and move the website over to the blowfish theme, but that's a task for another day
- I used Cloudflare already for a couple things like Dynamic DNS for my Unifi router, SSL certs from Let's Encrypt for my internal homelab services, and hosting my site previously with a hosting provider (I don't recommend this for most people because it's way too much money and faffing about with Wordpress). 
- Github Pages was an easy way to host a website and while I was already pretty familiar with Github in general, I never used their actions or static pages features. I felt this was a good way to learn and I also found that it supports Hugo by default, so it seemed like the obvious choice. 

## Getting Hugo Up & Running
The first *"challenge"* was getting Hugo working. There's a couple ways you can do this (not all of them necessary if you just plan on deploying with Github Pages), but if you want the ability to test out changes before pushing them to your official github repo, I recommend it. 

On top of that, despite the seemingly complicated process of building it from source, you don't have to go that route. What I did, following the [official documentation](https://gohugo.io/installation/), was I built it by installing the winget package (Choose your own package manager based on your preference and OS). All I did was open up Powershell 7 and run the following command:
```bash
winget install Hugo.Hugo.Extended
```
Once that's done, we can finally get into using Hugo to build our site.

## Building the Website
### Backbone
First steps here are to actually initialize our site. I placed my site files in my documents folders by default, but in general the creation of a site follows these commands:
```bash
hugo new site mysite
cd mysite
git init
```

Once that's done, we get to add a theme. This bit is a personal preference, but I personally like a site that stays out of my way and keeps the content plain to see. I also did not want to do too much work configuring a more complicated theme (like Blowfish), so I went with PaperMod. Simple, light, and quick with still plenty of customization. To add a theme you input these commands:
```bash
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```
I went with something much easier to use out of the box, called `PaperMod`, but it's all up to you which theme you want. The theme here with Hugo is customizability.

At this point, I would recommend an IDE to assist in editing your website files. Personally I use Visual Studio Code because it's what I'm most familiar with and it has plenty of plugins to make life easier (Ex: syntax highlighting, code autocomplete, easy file management, etc.)

Now we can get to customizing what the site actually shows us. For now, we can just use  "username.github.io" and just replace the "username" with your Github username. I'll be using the example of `princessdonut.github.io`. We also need to make sure we specify our theme. is going to be `PaperMod`. This is the theme we just added with our previous command. Things will break if you do not specify this. Ask me how I know... I know because it took me a long time to notice the problem... Like hours... Anyways! This file is usually in the toml language, which I find to be less readable than yaml, but since it's the default, we will stick with it. In our `hugo.toml` file, we'll add this: 
```toml
baseURL = "https://princessdonut.github.io/"
title = "My Site"
theme = "PaperMod"
languageCode = "en-us"

[params]
  defaultTheme = "auto"
  ShowReadingTime = true
  ShowShareButtons = true
```

The last part of our hugo site's backbone has to be our file that tells it to deploy with Github actions. The following file goes in `.github/workflows/hugo.yml` file:
```yaml
name: Deploy Hugo site to Pages

on:
  push:
    branches: ["main"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

defaults:
  run:
    shell: bash

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.128.0
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: Build with Hugo
        run: |
          hugo \
            --minify \
            --baseURL "${{ steps.pages.outputs.base_url }}/"
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

### Adding Content
At this point, we can now add content! We'll add a post first to our content folder. I personally like to make a subfolder called `posts/`. In fact I would get into this habit for most of the site save for something like a resume or an about page. 

This post is named hello-world.md, indicating that it is a markdown file. A post follows this general layout, depending on what you want:
```markdown
---
title: "Hello-world"
date: 2025-04-03
draft: false
---

This is my first post on my website. Hoping for many more
```

This is where we need to get used to markdown language. Markdown is a lightweight markup language created by John Gruber in 2004, designed to format plain text in a way that is easy to read and write. It allows users to add formatting elements like headings, lists, and links using simple symbols, making it popular for web development, documentation, and blogging.

We have some examples of this formatting here. You have your `title` which is just the title of the post, the `date` which (big surprise) is the date the post is made, `draft` which states whether this post should be shown or not, false means it's not a draft and is shown on the site, and true means it should not be shown on the site. The `---` symbols kind of indicate the metadata of the site and everything outside of that is the content of the post. That's just what's in this placeholder post. There are a lot of other modifiers you can add to text in markdown to make things look different.

If you haven't used the markdown language before I highly recommend it. I'll be making another post about how much I enjoy using Obsidian because it's a perfect app/language combo for note taking. Easy syntax and well documented and all while being regular text. That, however, is for another day.

## Testing
This next bit is also optional depending on what you want, but again, I recommend it since it is a good way to test things before committing it to your Github page. We will now deploy the hugo server locally by entering in this command in your hugo directory:
```bash
hugo server -D
```
You can now visit the website at `http://localhost:1313`

## Github Repo
Now for the part where we actually get the website up on the internet... sort of. This part is where knowledge on git is required since we'll be using it to deploy to Github. 

We are going to make a repo in Github to host all the files and changes we make to the website. We want to name the repo `<username>.github.io`. It has to be public (a limitation on the free version of Github pages) and we will not be adding any README, licenses, gitignore, or any other files to avoid conflicts. Let's say for example the github repo is at `https://github.com/princessdonut/princessdonut.github.io.git`

In the repo, we need to set some stuff up for Pages specifically. Go to the `Settings` menu, click on `Pages`, and make sure your Build and Deployment ection shows Github Actions as the source. This is also where we need to make sure HTTPS is enforced and where we can add a custom domain if we want. We'll go over that after we get the repo set up.

Now that the repo exists, we can push our code to it using the following command to do our initial commit. Make sure your `.github/workflows/hugo.yml` file is included in the first commit, otherwise nothing will deploy yet.
```bash
git remote add origin https://github.com/princessdonut/princessdonut.github.io.git
git add .
git commit -m "Initial commit"
git push -u origin main
```

## Did it Work?
We are now asking ourselves, did it work? Well, we can very easily check. First off, we check the website. It is up? Do our posts show up? If yes, great! Continue reading anyways. If not, we can check on what went wrong by looking at the Github repo and checking the `Actions` tab. This shows what Github is doing to present the website. I usually look at this while it's deploying out of habit to make sure things go right. 

## Custom Domain
I mentioned earlier that we can add a custom domain to this to really make it our own. Now that the website is actually up and running, we can safely move on to that. 

### DNS
This requires some prior knowledge on how to navigate your domain registrar's website, so I'll be as general as I can. First, go to Cloudflare. Next, we need to go to our dashboard, add a site, enter in our domain, follow our domain registrar's instructions for pointing our NS records to Cloudflare's assigned nameservers. For porkbun's instructions its `Domain settings → NS records → point to Cloudflare's assigned nameservers`. 

Next, we add the DNS records. Continuing with our example, we need to add the following to Cloudflare:

| Type | Name | Content | Proxy status |
|------|------|---------|--------------|
| A | @ | 185.199.108.153 | Proxied or DNS only |
| A | @ | 185.199.109.153 | Proxied or DNS only |
| A | @ | 185.199.110.153 | Proxied or DNS only |
| A | @ | 185.199.111.153 | Proxied or DNS only |

For www subdomain:
| Type | Name | Content | Proxy status |
|------|------|---------|--------------|
| CNAME | www | princessdonut.github.io | Proxied or DNS only |

One major concern here: Do *NOT* set this to `Proxied`. It routes the traffic through Cloudflare first, which sounds nice, but it interferes with Github's automatic HTTPS cert provisioning. Just keep it as standard `DNS Only`.

### Our TOML File
Finally, we just need to update our hugo.toml file. We will be updating the baseURL to our new website:
```toml
baseURL = "https://princessdonut.com/"
title = "My Site"
theme = "PaperMod"
languageCode = "en-us"

[params]
  defaultTheme = "auto"
  ShowReadingTime = true
  ShowShareButtons = true
```

Now we can push that change to Github and we just have to wait. DNS propagation can sometimes take up to 24 hours, realistically it took like 5 minutes when I was deploying my site. the beauty of Github Pages is that you do not need this last step. If you want, you can keep it like this and just have a .github.io page, but I wanted to go the extra mile.

## Final Thoughts
This obviously seems like a lot and it definitely is. There are a lot of different technologies at play here, but with the ease of use of Hugo, and the well documented process of getting a static website going, this is something that's easily attainable for most techy individuals.

Cheers!