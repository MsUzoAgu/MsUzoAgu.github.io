---
layout: post
title: A Simple Git Pre-Push Hook 
description: pre-push hook to check for changelog updates
summary: git pre-push hook reminder to update my changelogs
comments: true
tags: [git]
---

On my personal projects, the script below reminds me to update my changelog:

```shell
	#!/bin/sh
	exec < /dev/tty
	while true; do
		read -p "Have you updated the changelog? (y/n) " response
		
		if [[ "$response" ==  "y" ]] || [[ "$response" == "Y" ]]; then
			exit 0
		else 
			echo >&2 "Please run '$ codelog new <NAME>' to record the changes made"
			exit 1
		fi
	done
```

Script is placed in hooks subdirectory of the Git directory (**project_dir/.git/.hooks**).