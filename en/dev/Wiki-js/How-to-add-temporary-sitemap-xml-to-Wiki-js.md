---
title: Ad-hoc way to add sitemap.xml to Wiki.js
description: 
published: true
date: 2022-12-24T21:13:46.682Z
tags: cron, english, sitemap, wikijs
editor: markdown
dateCreated: 2022-12-24T21:09:21.809Z
---

- [Wiki.js에 sitemap.xml 을 추가하는 임시단편적인 방법***Korean** version of this document is available*](/ko/dev/Wiki-js/How-to-add-temporary-sitemap-xml-to-Wiki-js)
{.links-list}

> Since Wiki.js 3.0 will officially support the sitemap function, this is a temporary method.
> The method below assumes that access to assets in the OS project directory where Wiki.js is installed and use of crontab are possible.

---

- The wiki.js operating server environment is configured as follows.
   - wiki.js installation directory: `/home/wiki/app`
   - sitemap.xml creation script location: `/home/wiki/.scripts/wikijs-sitemap` (empty directory)
  
## Wiki.js pre-preparation

- Put links on the first page of your wiki.js so that you can browse all pages.
- Refer to the bottom of [Home](/home) in wiki.yowu.dev.

## sitemap.xml creation script

### initial setup

```bash
cd /home/wiki/.scripts/wikijs-sitemap
npm init -y
npm install --save https://github.com/uyu423/sitemap-generator
```

> There is an issue where the `sitemap-generator` project does not properly parse the wiki.js page.
> As a `cheerio` version issue used in the project, a pull request is currently in progress. [sitemap-generator#119](https://github.com/lgraubner/sitemap-generator/pull/119)
> If the `cheerio` of the project is up, you can use the original project.
> ```bash
> npm install sitemap-generator
> ```
> 😎
{.is-warning}

### create script (node.js)

```javascript
const SitemapGenerator = require("sitemap-generator");

const service = SitemapGenerator("https://your.wiki.address", {
  filepath: "/home/wiki/app/assets/sitemap.xml",
  stripQuerystring: true,
  priorityMap: [0.5, 0.5, 1.0]
});

service.start();
```

- Change `filepath` and `priorityMap` according to your own wiki.js configuration.
- When `node index.js` is executed, the `sitemap-generator` crawler recursively finds the `<a href>` tag in the page body and creates it as sitemap.xml.
- Check if sitemap.xml is normally created in the wiki.js assets directory.

## crontab

- I used `crontab`, a basic Linux utility, to update sitemap.xml once a day.

```bash
crontab -e
```
```cron
23 4 * * * node /home/wiki/.scripts/wikijs-sitemap/index.js
```
- Update `sitemap.xml` every day at 04:23 am

## It's over

- Now, register your sitemap with the address `https://your.wiki.address/_assets/sitemap.xml` in the search engine you need.
> Here, we shared an example of adding `sitemap.xml` to the server local environment in a very cumbersome way.
> If possible, it would be good to customize according to the server environment where Wiki.js is running.