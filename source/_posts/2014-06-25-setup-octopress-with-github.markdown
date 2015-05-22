---
layout: post
title: "Setup Octopress with GitHub"
date: 2014-06-25 15:36
comments: true
categories: [blog, octopress, github ]
---
This post will guide you how to setup the Octopress blog and host it with Github.
Why did I choosed Octopress for my blogging, I guess it was simple and better integration with github, and I didn't wanted to spend any extra bucks for this.
I'm currently setting this up in Elementary OS backed by ubuntu 12.04.

Check whether your ruby version is 1.9.3 or more then this
{% codeblock %} apt-get install ruby1.9.1-full {% endcodeblock %}

Clone the Octopress to your working directory.
{% codeblock %} git clone git://github.com/imathis/octopress.git octopress {% endcodeblock %}

<!-- more -->
Run this command and the Octopress is good to go.
{% codeblock %}
cd octopress/
sudo gem install bundler
bundle install
rake install
{% endcodeblock %}

Just to check whether every thing is working or not.
{% codeblock %} rake preview {% endcodeblock %}

Check in your local browser, whether you can view page or not.
{% codeblock %} http://localhost:4000 {% endcodeblock %}

If you can see the Octopress sample page. that means you are good to go. Next we can setup it with Github.
It will ask your repository link, in my case it is `git@github.com:ashish1099/ashish.github.io.git`
{% codeblock %}
rake setup_github_pages
rake generate
rake deploy
{% endcodeblock %}

Next we need to commit the source branch into GitHub.
{% codeblock %}
git add .
git commit -m 'your message'
git push origin source 
{% endcodeblock %}

If anything failed, check this commands.
{% codeblock %}
git remote -v
octopress      git@github.com:imathis/octopress.git (fetch)
octopress      git@github.com:imathis/octopress.git (push)
origin	git@github.com:ashish1099/ashish.github.io.git (fetch)
origin	git@github.com:ashish1099/ashish.github.io.git (push)
{% endcodeblock %}

If the above command is not returning your git repository, then probably you will have to add this.
{% codeblock %}
git remote add origin git@github.com:ashish1099/ashish.github.io.git
{% endcodeblock %}

Check what is the branch, it should be `source` branch.
{% codeblock %}
git branch
* master
{% endcodeblock %}

If it is not `source` branch, then change it to `source` branch by this command.
{% codeblock %}
git branch -m master source
git branch
* source
{% endcodeblock %}
