---
layout:     post
title:     	git log format
subtitle:    "\"git log format\""
date:       2017-12-29
author:     Mr Chang
header-img: img/post-sample-image.jpg
catalog: true
tags:
    - git log format

---



默认git log 出来的格式并不是特别直观，很多时候想要更简便的输出更多或者更少的信息，这里列出几个git log的format。
可以根据自己的需要定制。

git log命令可一接受一个--pretty选项，来确定输出的格式.

比如 ： 

如果我们只想输出hash.

	git log --pretty=format:"%h" 
	

详细 命令 ： 


'%H': commit hash

'%h': abbreviated commit hash

'%T': tree hash

'%t': abbreviated tree hash

'%P': parent hashes

'%p': abbreviated parent hashes

'%an': author name

'%aN': author name (respecting .mailmap, see git-shortlog or git-blame)

'%ae': author email

'%aE': author email (respecting .mailmap, see git-shortlog or git-blame)

'%ad': author date (format respects --date= option)

'%aD': author date, RFC2822 style

'%ar': author date, relative

'%at': author date, UNIX timestamp

'%ai': author date, ISO 8601-like format

'%aI': author date, strict ISO 8601 format

'%cn': committer name

'%cN': committer name (respecting .mailmap, see git-shortlog or git-blame)

'%ce': committer email

'%cE': committer email (respecting .mailmap, see git-shortlog or git-blame)

'%cd': committer date (format respects --date= option)

'%cD': committer date, RFC2822 style

'%cr': committer date, relative

'%ct': committer date, UNIX timestamp

'%ci': committer date, ISO 8601-like format

'%cI': committer date, strict ISO 8601 format

'%d': ref names, like the --decorate option of git-log

'%D': ref names without the " (", ")" wrapping.

'%e': encoding

'%s': subject

'%f': sanitized subject line, suitable for a filename

'%b': body

'%B': raw body (unwrapped subject and body)

'%N': commit notes

'%GG': raw verification message from GPG for a signed commit

**出自 ： https://git-scm.com/docs/pretty-formats**
