+++
date = 2020-01-26T06:30:00Z
disable_comments = true
tags = ["CI/CD", "github", "webhooks", "hugo"]
title = "Setting up this blog - Part 2"
type = ""

+++
In this post we will see how to setup Hugo and leverage GitHub webhooks to auto-run Hugo whenever a commit is pushed to the GitHub repository. The basic process goes like this

* I commit a new post as a Markdown post into the GitHub repository
* GitHub triggers a webhook that I have setup on every push that's done into the repository
* The webhook endpoint (running on the same server as the blog) runs a bash script locally to trigger Hugo build. The new post is now live.
* The bash script also pushes the Hugo updated public HTML files into the repository for next build run.

***

### PART 2

#### Hugo installation and theme setup

SSH into the server and then install Hugo on the server. Obtain the link to the latest release build from [https://github.com/gohugoio/hugo/releases](https://github.com/gohugoio/hugo/releases)

```bash
wget https://github.com/gohugoio/hugo/releases/download/v0.62.0/hugo_extended_0.62.0_Linux-64bit.deb
sudo dpkg -i hugo_extended_0.62.0_Linux-64bit.deb
```

You can find the Hugo config.toml file from my website in this [gist](https://gist.github.com/patali/f020aca2ef2647a1b979695348209222). Notice the parameters that is configured for staticman, we will discuss that in depth in the later posts.

I also modified the awesome Ezhil theme to support staticman commenting , error page customization and a custom dark theme. You can find my repository for the modified theme [here](https://github.com/patali/ezhil)

#### Setting up Endpoint

For build and deploy automation to work. We make use of GitHub's webhooks facility. First we need to set up an endpoint that GitHub can hit every time it receives a commit. After a bit of researching I decided to go with Adnan's awesome Webhook project [https://github.com/adnanh/webhook](https://github.com/adnanh/webhook). It seemed perfect for my use case.

Installing Webhook

```bash
sudo apt install webhook
```

Create the **hooks.json** file containing all the exposed endpoints and commands that needs to be executed. This is the hook file that I wrote to handle GitHub triggered events. Webhook also matches the signature so that the endpoint only works when triggered by GitHub.

```json
[
  {
    "id": "web_hook_endpoint_name",
    "execute-command": "/home/ubuntu/deploy.sh",
    "command-working-directory": "specify-the-working-directory",
    "trigger-rule": {
      "and": [
        {
          "match": {
            "type": "payload-hash-sha1",
            "secret": "SECRET-KEY-THATS-SPECIFIED-AT-GITHUB-WEBHOOK-CONFIG",
            "parameter": {
              "source": "header",
              "name": "X-Hub-Signature"
            }
          }
        }
      ]
    }
  }
]
```

The **deploy.sh file** looks like this and it resides in my blog's local git repository folder

```bash
!/bin/sh
git push
git pull origin master
hugo
git add .
git commit -m "Hugo: Auto generated public file changes"
git push origin master
```

Run webhooks with the hooks.json file that we created above. Keep in mind that if you installed webhooks from apt then it might be already running as a service. I suggest that you kill it before starting again.

```shell
sudo killall webhook
webhook -hooks hooks.json -verbose
```

Once that is setup we need to choose an endpoint URL. I created a subdomain from my domain registrar's control panel and setup the Nginx reverse proxy configuration for it. Also used Certbot to generate the certificate for this subdomain. This is the basic reverse proxy configuration that I used to point all traffic incoming to the webhook process running at port 9000

```nginx
server {
        server_name deploy.patali.in;
        index index.html;

        location / {
                proxy_pass http://localhost:9000/hooks/web_hook_endpoint_name;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Forwarded-Proto $scheme;

                client_max_body_size 0;
                add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload";
                add_header Referrer-Policy "same-origin";

                access_log /var/log/nginx/webhook-deploy-patali.access.log;
                error_log /var/log/nginx/webhook-deploy-patali.error.log;
        }
}
```

 
 

#### Setting up GitHub to trigger the webhook

* Go to to your repository's settings page and select Webhooks from the left panel.
* Click on Add webhook
* Enter the endpoint that you want to be triggered
* In the secret section make sure you enter the same secret that you used in your hooks.json file

![](https://patali-blogs-bucket.s3.us-east-2.amazonaws.com/patali.in/webhookgithubsetup.jpg)

That's all. Now every time a commit is pushed to the repository the deploy.sh script get triggered automatically and the new site changes becomes live instantly. In the next part of this blog series we will learn about integrating Staticman commenting system into a Hugo generated static website.

***

Quick links to other parts in this series  \
[Part 1](https://patali.in/posts/setting-up-this-blog-part-1/) | [Part 2](https://patali.in/posts/hugo-and-ci-using-github-webhook/) | [Part 3](https://patali.in/posts/setting-up-this-blog-part-3/) | [Part 4](https://patali.in/posts/setting-up-this-blog-part-4/)

***