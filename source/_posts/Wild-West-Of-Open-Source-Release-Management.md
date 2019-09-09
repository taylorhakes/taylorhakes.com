title: Wild West of Open Source Release Management
permalink: wild-west-of-open-source-release-management
date: 2019-09-08
---

Open source code is an amazing part of software development. There is a small group of lucky people that get to work on open source software full time, either paid by their company, work on a project that is usefull enough that it receives enough donations, or just financially stable enough personally to do it. The reality is that most people work on open source as a side project, either after work hours, during lunch etc. They devote some portion of their free time to working on a project that they make little or no money from. It's hard to enumerate all the benefits free open source has brought the software development community. I think most people that use open source software know and understand them well.

With all the benefits, I think it's sometimes good to talk about issues, so we can try to address them. One issue I have been thinking about recently is with release management. The process from taking a version of open source code in git/mecurial/etc. and releasing it as a consumable piece of software either for direct download, though an app store or a package manager, etc. 

One of the benefits of open source software is that it is auditable by any person that wants to take a look at it. The release process unfortunately is not auditable on most projects. Often the process is undefined or unknown for open source projects. This is especially true for smaller projects. Generally, one of the contributors of the project will create a release on their laptop to make it available for download. More sophisticated projects will use a shared build server. In either case, the problem is that it is tough to determine if the release code is the same as the code in the repo. This can happen because the code goes from a readible source code form to a binary that is less human readable. This can even happen in JS where code is often minified and obfuscated, making it harder to read the code.

So what's the problem? There are many security related issues that can happen during the release process. A malicious maintainer, a hacker with leaked credentials or a virus on a maintainers laptop can insert code during the release process. Most projects have no tools or processes to catch the issue even after the fact. In the past, most issues are caught by the end user.  Sometimes days or weeks after the issue occurs. This is a serious problem on open source projects like password managers.

I want to be clear. I am not blaming open source projects for these issues. They have limited or no funds and very little time

Some open source projects have been able to create reproducible builds. This means that anyone can follow the build steps and create a bit for bit copy of the release. This is a great step in the right direction, but very few projects have been able to create reproducible builds. 


<!-- more -->