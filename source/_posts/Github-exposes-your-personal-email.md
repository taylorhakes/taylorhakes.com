title: Github exposes your personal email address
permalink: get-any-github-users-email-address
date: 2016-02-07
---
Github by default exposes your personal or work email in GIT history. It is not visible on the Github website, but easily available from the command line. 

![git-log](/img/git-log.png)

This is very unfortunate default. Github recognizes this as an issue and has a [page dedicated](https://help.github.com/articles/keeping-your-email-address-private/) to ways to change the default. I recommend everyone follow the steps in the article and anonymize their email. Unfortunately, there is no reasonable way to modify old commits with your private email. Github suggests modifying GIT history, but this is not an option on any public repos or any repo you do not have control over.

Hopefully Github can change this default behavior and figure out a way to hide existing email addresses.