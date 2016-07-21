---
layout: post
title:  "Git mailmap"
date:   2016-07-21
tags: []
categories: Personal
---

# Using mailmap to fix authors list in git

I love doing `git shortlog -sne` to get number of commits by each author in a git repo ( don’t aske me why!)

It's a useful command to check commits. But when you on many computers maybe have many email setting for Git.

That's why .mailmap file been design.

It's simply use. Just put .mailmap file on your root directory.

Then code likes this.

```
bodyno <az8321550@gmail.com>  <411298027@qq.com>
bodyno <az8321550@gmail.com>  <az8321550@gmail.com>
bodyno <az8321550@gmail.com>  <az8321550@hotmail.com>
bodyno <az8321550@gmail.com>  <az8321550@gmai.com>
```

Then run `git shortlog -sne` again.

Look what happens.

Thanks everyone.