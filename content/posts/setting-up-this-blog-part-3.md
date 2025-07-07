+++
date = 2020-01-26T06:52:00Z
disable_comments = false
tags = ["static", "github", "staticman"]
title = "Setting up this blog - Part 3"
type = ""

+++
In this post we are going to tackle the problem of _How do I enable a commenting system on a static html website?_

I learned that there are basically two ways to achieve this

1. Use an external service like Disqus, Commento etc
``` text
Pros:
  * Super easy to setup
  * No Self hosting necessary
Cons:
  * User privacy/tracking issues
  * Data is not hosted on your server
  * Many other things
```
2. Use [Staticman](https://staticman.net/)
```
Pros:
  * Self hosted
  * No visitor tracking involved
  * Comments are stored directly as a static html
Cons:
  * Relatively complex setup
  * Self hosted <hehe>
```
This year my goal has been to learn as many new things as possible. I'm also deeply concerned about privacy issues so Staticman was a natural choice for me.

So a basic commenting process on Staticman works in this way

* Runs as a web application on the server and exposes an url to submit the comment from the html form
* A new GitHub user is registered & that user is given write permission to the website's repository.
* Staticman uses this new GitHub user's credentials to create pull requests on the main repository every time a new comment arrives. This enables comment moderation.
* You can then accept or reject the pull requests to add the comment to the static html page.

***

### Part 3

#### Setup alternate GitHub user account and Access Tokens

Firstly setup a new GitHub account and generate access tokens

* Register a new account on GitHub and login.
* Navigate to [Settings > Developer Settings > Personal Access Token page](https://github.com/settings/tokens)
* Click on "Generate new token" button. Provide a note and give it "repo" privileges and click "Generate token"

**Make sure you note down the token as you wont be able to see it again.**

![](https://patali-blogs-bucket.s3.us-east-2.amazonaws.com/patali.in/GithubPersonalAccessToken.jpg)

Next generate a private key (using ssh-keygen or openssl) on the server and add it to [GitHub SSH Keys](https://github.com/settings/keys). Let's call the key as **staticman-private-key** as we will be using it later in Staticman configuration.

After this add the newly created GitHub account as a collaborator to the Website's main repository.

#### Install and deploy Staticman

Clone the Staticman repository into a local directory on the server

```shell
git clone git@github.com:eduardoboucas/staticman.git
```

Open **docker-compose.yml** file inside the clone folder and update

* GITHUB_TOKEN : Use the access token that you generated above
* RSA_PRIVATE_KEY : Use full content of **staticman-private-key** within ""

So the file should look something like this after modification

```yaml
    version: '2'
    services:
      staticman:
        build: .
        env_file: .env
        ports:
          - '1111:3000'
        restart: unless-stopped
        environment:
          NODE_ENV: production
          PORT: 3000
          GITHUB_TOKEN: abcdefghijklmnopqrstuvwxyz
          RSA_PRIVATE_KEY: "-----BEGIN RSA PRIVATE KEY-----\nXXXXXXX"
```

Register a subdomain name and setup Nginx to forward all requests to **port 1111** as configured above. Check [Part 2](https://patali.in/posts/hugo-and-ci-using-github-webhook/) of this series on how to setup configure Nginx reverse proxy.

Run Staticman

```shell
docker-compose up
```

You now have the commenting system back-end ready.

#### Integrating Staticman into Hugo & template modification

You need to create a staticman.yml file in the root of the website's repository. This file essentially specifies what fields are used and which fields are mandatory for comment submission. My staticman.yml file looks like this

```yml
---
comments:
    allowedFields:
        - name
        - comment
    branch: master
    commitMessage: "New comment in {options.slug}"
    filename: "comment-{@timestamp}"
    format: yaml
    generatedFields:
        date:
            options:
                format: iso8601
            type: date
    moderation: true
    path: "data/comments/{options.slug}"
    requiredFields:
        - name
        - comment
```

You will also have to specify some additional parameters in the website's config.toml file
Here are the parameters that are in my config.toml file

```yml
[params.staticman]
branch = "master"
endpoint = "https://subdomain.patali.in/v2/entry"
repository = "patali.in"
username = "github_user_name"
```

Next to display the comments that are approved. We need to modify the Hugo template. In this site I have modified Ezhil theme to my needs. Firstly I created a partial called comments.html

```html
{{ if and .Site.Params.staticman (not (or .Site.Params.disable_comments .Params.disable_comments)) }}
   <section id="comments">
       {{ if .Site.Params.staticman }}
       <h3 class="title"><a href="#comments">&#9997; Comments</a></h3>
       <section class="staticman-comments post-comments">
           {{ $comments := readDir "data/comments" }}
           {{ $.Scratch.Add "hasComments" 0 }}
           {{ $postSlug := .File.BaseFileName }}
   
           {{ range $comments }}
           {{ if eq .Name $postSlug }}
               {{ $.Scratch.Add "hasComments" 1 }}
               {{ range $index, $comments := (index $.Site.Data.comments $postSlug ) }}

                   <div id="commentid-{{ ._id }}" class="post-comment">
                       <div class="post-comment-header">
                           <p class="post-comment-info">
                               <span class="post-comment-name">{{ .name }}</span>
                               <br>
                               <a href="#commentid-{{ ._id }}" title="Permalink to this comment">
                                 <time class="post-comment-time">{{ dateFormat "Jan 2, 2006 at 15:04 MST" .date }}</time>
                               </a>
                           </p>
                       </div>
                       <div class="post-comment-message">
                           {{ .comment | markdownify }}
                       </div>
                   </div>
                   
           {{ end }}
       {{ end }}
   {{ end }}

       <form id="comment-form" class="post-new-comment" method="post" action="{{ .Site.Params.staticman.endpoint }}/{{ .Site.Params.staticman.username }}/{{ .Site.Params.staticman.repository }}/{{ .Site.Params.staticman.branch }}/comments">
           <h3 class="prompt">Say something</h3>
           <input type="hidden" name="options[redirect]" value="{{ .Permalink }}#comment-submitted">
           <input type="hidden" name="options[slug]" value="{{ .File.BaseFileName }}">
           <input type="text" name="fields[name]" class="post-comment-field" placeholder="Name *" required/>
           <br />
           <textarea name="fields[comment]" class="post-comment-field" placeholder="Comment (markdown is accepted) *" required rows="5" cols="50"></textarea>
           <br />
           <input type="submit" class="comment-submit-button" value="Submit">
       </form>
   </section>
   {{ end }}
</section>
{{ end }}
```

I then included this partial in all the pages of the template that I wanted commenting. In my case I needed it only on pages and posts : single.html You can check out the modified theme here: [https://github.com/patali/ezhil](https://github.com/patali/ezhil "https://github.com/patali/ezhil")

We are done! The commenting system should work now as described in the beginning of this post. 

The next post is going to be the final post in this series. I'm going to briefly explain what settings I use in forestry.io to make it work well with changes I did to my Hugo template.

***

Quick links to other parts in this series  \
[Part 1](https://patali.in/posts/setting-up-this-blog-part-1/) | [Part 2](https://patali.in/posts/hugo-and-ci-using-github-webhook/) | [Part 3](https://patali.in/posts/setting-up-this-blog-part-3/) | [Part 4](https://patali.in/posts/setting-up-this-blog-part-4/)

***