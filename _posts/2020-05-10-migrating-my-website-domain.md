---
layout: post
title: "Migrating my website domain"
author: Julio Trigo
date: 2020-05-10 22:00:00 +0100
last_modified_at: 2020-05-10 22:00:00 +0100
permalink: /articles/migrating-my-website-domain/
tags:
  - domain names
  - HTTPS
  - DNS
  - GitHub Pages
  - Netlify
---

I have been using the `blog` subdomain for this website since I started my blog around 2013 (on [Blogger](https://www.blogger.com), before I moved to [GitHub Pages](https://pages.github.com/)). I did not want to build something myself for publishing my articles and Blogger gave me the option of using my own custom domain. Since I was already using `www` on my main website, I decided to go with `blog` instead.

<!--more-->

I have wanted to consolidate my websites in one domain name for a long time to have all my content available in a single place. This is also normally recommended for SEO.

However, it wasn’t until I read about [Webmention](https://www.w3.org/TR/webmention/), [IndieWeb](https://indieweb.org/), [POSSE](https://indieweb.org/POSSE), and [Web sign-in](https://indieweb.org/Web_sign-in) that I decided to finally go ahead and move all my content under my primary domain: if I am going to use my personal web address as my ID then I need to decide what domain name I want to employ for that.

## Should I use www?

So the first step was to decide the primary domain for my website: `www.juliotrigo.com`.

[Here](https://www.yes-www.org/why-use-www/) you can read about why it is recommended to use `www` instead of *apex* domains (also called *bare*, *naked*, or *root* domains) as the primary domain, which should be part of the [canonical website URLs](https://www.mattcutts.com/blog/seo-advice-url-canonicalization/).

## Migration changes with GitHub Pages

There is some documentation about [configuring](https://help.github.com/en/github/working-with-github-pages/configuring-a-custom-domain-for-your-github-pages-site) and [managing](https://help.github.com/en/github/working-with-github-pages/managing-a-custom-domain-for-your-github-pages-site) custom domains with GitHub Pages out there.

These are the changes I made:

* **Code** (diff):

```yaml
# Jekyll _config.yml

-url: "https://blog.juliotrigo.com"
+url: "https://www.juliotrigo.com"

# I have started with play with https://www.w3.org/TR/webmention/
webmentions:
-  username: blog.juliotrigo.com
+  username: www.juliotrigo.com
```

```
# CNAME

-blog.juliotrigo.com
+www.juliotrigo.com
```

* **DNS**:

```
# DNS config

DNS ENTRY       TYPE            DESTINATION/TARGET
@               A               185.199.108.153
@               A               185.199.109.153
@               A               185.199.110.153
@               A               185.199.111.153
blog            CNAME           www.juliotrigo.com.
www             CNAME           my-subdomain.github.io.
```

### GitHub Pages limitations

GitHub Pages had not been providing me all the freedom that I needed lately. For instance, by [restricting the Jekyll dependency versions that I can use](https://pages.github.com/versions/).

Additionally, I had a few [HTTPS issues](https://github.community/t5/GitHub-Pages/Does-GitHub-Pages-Support-HTTPS-for-www-and-subdomains/td-p/7116) with my apex domain after the migration, as explained [here](https://github.community/t5/GitHub-Pages/Does-GitHub-Pages-Support-HTTPS-for-www-and-subdomains/m-p/7202#M495):

> GitHub doesn’t currently support creating a certificate that covers both your root domain and your `www` subdomain. We only generate a certificate for the exact domain that you specify in the custom domain input box. (2018-05-10)

I also wanted to redirect the old subdomain (`blog`) to the primary one (`www`).

Even if there’s already a way of solving some of those things, I decided to give [Netlify](https://www.netlify.com/) a go.

## Migrating from GitHub Pages to Netlify

It was extremely simple to have it all set up with Netlify.

As part of this process, I had to ensure that:
* The website was **up and running** with no errors
* **HTTPS** was enabled and working
* All the relevant URLs were **redirected** to my primary domain

### 1) Set up, build and deploy the website

It was done with just a few simple steps:
* Sign up on Netlify
* Select the source code repository (*no code changes were required*)
* Create a new site
* Verify/amend the site settings (*the default values were good*)

```yaml
Build command: jekyll build
Publish directory: _site/
```

* Build and deploy the website

### 2) HTTPS setup

* **Netlify**:

```
my-subdomain.netlify.app: Default subdomain
www.juliotrigo.com: Primary domain
juliotrigo.com: Redirects automatically to primary domain
blog.juliotrigo.com: Domain alias
```

```
Your site has HTTPS enabled
Domains: blog.juliotrigo.com, juliotrigo.com, www.juliotrigo.com
Auto-renews in 3 months
```

* **DNS**:

```
# DNS config

DNS ENTRY       TYPE            DESTINATION/TARGET
@               A               104.198.14.52
blog            CNAME           www.juliotrigo.com.
www             CNAME           my-subdomain.netlify.app.
```

### 3) Fix redirects

Add [redirect rules](https://docs.netlify.com/routing/redirects/):

```apache
# _redirects

# Redirect default Netlify subdomain to primary domain
https://my-subdomain.netlify.app/* https://www.juliotrigo.com/:splat 301!

# Redirect old subdomain to primary domain
https://blog.juliotrigo.com/* https://www.juliotrigo.com/:splat 301!
```

[Include](https://jekyllrb.com/docs/configuration/options/) the `_redirects` file in the folder where the generated site will be placed (`_site`):

```yaml
# Jekyll _config.yml

include: [_redirects]
```

### 4) Verify all the relevant URLs

Check that all the relevant URLs are redirected to my canonical URL: `https://www.juliotrigo.com`.

```
# www subdomain
http://www.juliotrigo.com
http://www.juliotrigo.com/
https://www.juliotrigo.com
https://www.juliotrigo.com/

# Apex domain
http://juliotrigo.com
http://juliotrigo.com/
https://juliotrigo.com
https://juliotrigo.com/

# The old blog subdomain
http://blog.juliotrigo.com
http://blog.juliotrigo.com/
https://blog.juliotrigo.com
https://blog.juliotrigo.com/

# Netlify default subdomain
http://my-subdomain.netlify.app
http://my-subdomain.netlify.app/
https://my-subdomain.netlify.app
https://my-subdomain.netlify.app/
```

## Things I've learned

* The benefits of using `www` as the primary domain
* The functionality that Netlify provides on their *Starter* free plan is quite powerful and simple to use
* The SSL/TLS certificate that Netlify issues can cover extra subdomains (free of charge)
* The `_redirects` file is a simple way of doing HTTP redirects
