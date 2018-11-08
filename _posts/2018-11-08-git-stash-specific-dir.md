---
layout: post
title: "git 팁들 작성중"
description: "git"
date: 2018-11-08
tags: [git, stash, tip]
comments: true
share: false
    
---





* 특정 폴더나 파일만 `git stash`하고 싶을 때
  * 그 파일이나 폴더를 제외한 모든 변경사항을 `git add` 하기
  * `git stash --keep-index`
  * `git reset`