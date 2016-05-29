title: Fix Github Tab Formatting
date: 2016-04-18 21:14:57
---
If you use tabs and commit your code to Github, you probably are annoyed at how Github formats your code. Github, by default, makes tabs appear as 8 spaces.


![Github Tabs](/img/github-tabs.png)


There may be a few people who use this style, but the majority of people have their editor make tabs appear as 4 or 2 spaces. Luckily there is a solution that I stumbled upon recently. Add an [.editorconfig](http://editorconfig.org/) file to your repo at the top level.

If you haven't used an [.editorconfig](http://editorconfig.org/), it essentially is a file that tells your editor how to indent code, end lines, end the file and which character set to use. Even if you don't want to change your Github formatting, it is a very useful file. People are less likely to commit badly formatted code. Most editors support .editorconfig out of the box or with plugins.

Here is the .editorconfig that I use to make tabs look like 2 spaces.


![.editorconfig](/img/editorconfig.png)

With the above .editorconfig committed to the repo, the code now looks like this on Github.


![Github Tabs Spaces](/img/github-tabs-spaces.png)