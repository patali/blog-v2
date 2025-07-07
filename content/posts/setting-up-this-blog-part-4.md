+++
date = 2020-01-26T07:17:00Z
disable_comments = false
tags = ["forestry.io", "s3", "CI/CD", "github", "cms"]
title = "Setting up this blog - Part 4"
type = ""

+++
In this final post about the infrastructure behind this website I'm going to write about how I manage and write posts without the need of using git commands.

I must admit that I started this site with an intention of not using any CMS as the posts are simple Markdown files, but as I started thinking more about the regular use case, having to git commit + git push is going to be hard especially when I'm using mobile devices (not impossible but cumbersome as I do use terminal emulators on my phone for work in the worst case scenarios x)).

My search for a CMS for static html blog ended with two options

* Netlify
* [forestry.io]()

I tried both but I liked the simplicity of forestry.io, so I decided to go with that. There was one more big reason why I chose forestry.io over Netlify but I cant remember anymore, this is the reason why I must document more haha.

***

### PART 4

Importing the project to forestry.io was super simple. After that I made some simple additions/changes to my forestry.io settings to make my life easy. Here are list of simple mods I did

#### Adding config.toml to Sidebar

Once imported I first added my root config.toml to the Sidebar. This enabled me to modify the basic blog settings from forestry UI

To do this. Navigate to Settings > Sidebar  and click on "Add section". Enter the following details

![](https://patali-blogs-bucket.s3.us-east-2.amazonaws.com/patali.in/forestry-config-toml.jpg)

#### Setting up Front matter Template

Front matter is the top portion of the markdown files that Hugo uses to set some variables like Date posted, Comments enabled/disabled, Title etc.

In my modified template I use some special fields like

1. title \[Text Field\] : Title of the page/post
2. tags \[Sortable List\] : Tags in that page/post
3. date \[Date Field\] : Date and time when the post was written
4. type \[Text Field\] : Defines where a page or a post
5. disable_comments \[Toggle\]: For enabling/disabling comments in posts/pages

Only Date needed a-bit of special formatting to make it work correctly with my template. I used **YYYY-MM-DDThh:mm:ssZ** as the Export format

![](https://patali-blogs-bucket.s3.us-east-2.amazonaws.com/patali.in/forestry_front_matter.jpg)

#### S3 Bucket for media storage

It's pretty convenient to use an S3 bucket to directly upload media files from forestry UI. Here's a link from forestry's documentation on how to setup an S3 bucket [https://forestry.io/docs/media/s3/](https://forestry.io/docs/media/s3/ "https://forestry.io/docs/media/s3/")

Here are my settings :P

![](https://patali-blogs-bucket.s3.us-east-2.amazonaws.com/patali.in/forestry-s3-settings.jpg)

#### Fin.

This entire project of setting up the blog has been quite interesting. I got an opportunity to learn many new tech and processes. But I must admit It feels like there are a-lot of moving parts and there might be other infrastructures which are much simpler that you can use and achieve the same results. 

My intention of this project has been solely educational. I wanted to learn CI/CD via webhooks, use Markdown for quick documentation and blogging, learn Nginx configurations. I think I was able to achieve all those goals. 

Thanks for reading.

***

Quick links to other parts in this series  \
[Part 1](https://patali.in/posts/setting-up-this-blog-part-1/) | [Part 2](https://patali.in/posts/hugo-and-ci-using-github-webhook/) | [Part 3](https://patali.in/posts/setting-up-this-blog-part-3/) | [Part 4](https://patali.in/posts/setting-up-this-blog-part-4/)

***