+++
date = 2020-01-25T06:30:00Z
disable_comments = false
tags = ["tech", "blogging", "github", "webhook", "staticman", "hugo"]
title = "Setting up this blog - Part 1"
type = ""

+++
Having got back to blogging after a decade it was rather an interesting experience researching on the available options for writing online. My old blog was a free WordPress website, it served me well back then but now I wanted to try something different. I had a few requirements

* I wanted to self host
* Needs to be a lightweight engine
* I should be able to easily write anywhere (preferably Markdown)
* Get to learn new things in the process of setting up

I looked at Medium, it's clean aesthetically and it's quick to write and post using their apps. But I not a fan of their content copyright policy and also not self hosted. I rejected WordPress as its heavy. After a bit of researching the best case scenario that meets all my requirements was to use a static site generator and self host a HTML site. However there were some limitations to this approach. For example no commenting forms and no CMS. But like any problem there are always some good solutions already available on the internet and some bad ones too x)

Finally this is setup that I decided to go with

* [Hugo](https://gohugo.io/) for static site generator
* Github for storage and [webhook](https://github.com/adnanh/webhook) for triggering Hugo builds
* [staticman](https://staticman.net/) for commenting
* [forestry.io](https://forestry.io/) for CMS

In the upcoming 3 or 4 blog posts I will be documenting how my setup looks like. I'm mostly documenting this for myself as otherwise I will end up forgetting how I set it all up eventually x), Hopefully it will be helpful for someone else on the same journey as me.

***

## Part 1

#### Setting up VPS and domain

I host this website on a $5 USD Droplet from DigitalOcean. Once the droplet is setup. I went through the usual server [hardening](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-18-04 "hardening") process of setting up firewalls and disabling password ssh etc etc. There are numerous articles written on proper server security practices online, so I went through tens of them to follow the strict processes.

I then purchased the domain name from name.com and pointed the root domain to the IP address of my droplet.

#### Setting up Nginx and Certbot

I use Nginx for all my web server and reverse proxy requirements. Also I use [Certbot](https://certbot.eff.org/) for https certificate generation and auto renewing. Lets begin with Nginx

Install Nginx from apt

```shell
 sudo apt install nginx
```

Create a new file Nginx config file etc/nginx/sites-available/{your domain name without www. For example patali.in}

Use this basic configuration to host a static HTML website

```nginx
server {
        server_name patali.in;

        index index.html;
        root /path-to-files/patali.in/public/;
        error_page 404 /404.html;

        location / {
                try_files $uri $uri/ =404;
                access_log /var/log/nginx/patali.in.access.log;
                error_log /var/log/nginx/patali.in.error.log;
        }
        location /404.html {
                root /path-to-files/patali.in/public/;
                internal;
        }
}
```

Point the **root** parameter to the directory were the Hugo generated HTML files resides on the server.

Symlink the configuration to Nginx's sites-enabled folder

```shell
sudo ln -s /etc/nginx/sites-available/patali.in /etc/nginx/sites-enabled/patali.in
```

Reload and reboot Nginx to load the new configuration

Install and run Cerbot

```shell
sudo certbot --nginx
```

Choose your domain from the list as the domain to generate certificates for and enable auto direction of all http to https traffic if you require it. Certbot will add the necessary extra Nginx configurations for https traffic automatically. It will also auto renew the certificates when it's about to expire.

That's it! This is all the setup you need to self host a static HTML website. In the next post I will be discussing about my Hugo based CI setup using GitHub webhooks.

Let me know if find you anything wrong with my setup or the instructions. I'm pretty new to this, so would love you hear from you. So please feel free to leave a comment or send me a [tweet](https://twitter.com/sharathpatali).

***

Quick links to other parts in this series    
[Part 1](https://patali.in/posts/setting-up-this-blog-part-1/) | [Part 2](https://patali.in/posts/hugo-and-ci-using-github-webhook/) | [Part 3](https://patali.in/posts/setting-up-this-blog-part-3/) | [Part 4](https://patali.in/posts/setting-up-this-blog-part-4/)

***