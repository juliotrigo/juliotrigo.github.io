---
layout: post
title: "Lato 2.0 Font Family"
date: 2019-11-12 23:47:00 +0100
last_modified_at: 2020-04-18 11:55:00 +0000
permalink: /posts/lato-2-0-font-family/
comments: true
tags:
  - lato
  - css
  - font-family
  - web-design
---

As part of a small redesign on this website, I wanted to use a high-quality and elegant font that *also looked good with text in italics* and **displaying content in bold**, but ***specially when combining both styles, which is something more difficult to find***.

After doing some research and comparing a few different fonts, I started to use the *Lato* [Google font](https://fonts.google.com/specimen/Lato). However, even though it looked quite good, I was not fully convinced.

<!--more-->

After a few hours comparing different fonts, it became quite hard for me to spot differences between them so I called it a day. The next morning, a bit fresher, I decided to know a bit more about *Lato* so I went to its [website](https://www.latofonts.com/lato-free-fonts/) to immediately see ***how beautiful*** the font looked on that page.

That font was different to the one I was currently using from Google Fonts, so I checked the content of that page and discovered a few interesting things:
* The *Lato* website was using the `LatoLatinWeb` font family
* There's a `2.0` (`2.015`) version of the *Lato* font family avail­able as a free down­load under the [SIL Open Font License 1.1](https://scripts.sil.org/cms/scripts/page.php?site_id=nrsi&id=OFL)
* It is specified that:
> The older ver­sion (1.0) of the Lato font fam­ily is avail­able on Google Fonts. We have no infor­ma­tion when Lato 2.0 will be avail­able on Google Fonts

This GitHub [issue](https://github.com/google/fonts/issues/6) (opened on the 9th April 2015) explains why *Lato* `2.0` is still not available on Google Fonts. It seem that *Lato* `2.015` uses a newer version of `ttfauothint`, which now produces a different result. Apparently, there was a deployment of that font version on Google Fonts in January 2017, but it had to be rolled out because it affected quite a few users ([Overnight my Lato looks quite different](https://github.com/google/fonts/issues/644)).

At that point, and taking all things into consideration, I decided that I still wanted to use the `2.0` version of the *Lato* font family on my website. This is how you can do it:
* Download the `Lato2OFLWeb` [Webfonts](https://www.latofonts.com/download/Lato2OFLWeb.zip)
  * In my case:`Version 2.015`
* Follow the instructions included in the `README-WEB.txt` file
* Select the font styles to be used
  * In my case: `LatoLatin-Regular`, `LatoLatin-Italic`, `LatoLatin-Bold`, and `LatoLatin-BoldItalic` from the `LatoLatin` group
* Copy the selected font style files to the website location where the static content is served from
* Inspect the `latolatinfonts.css` style sheet and use the CSS rules for the selected font styles
* Use `LatoLatinWeb` as a `font-family`
* Credit the authors of the *Lato* font for the original creation
  * In my case, done in the [License & copyright]({{ site.baseurl }}{% link license-copyright.md %})
 page

Finally, just enjoy the `2.0` version of the *Lato* font family!
