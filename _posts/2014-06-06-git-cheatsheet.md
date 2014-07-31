---
title: Useful git commands 
layout: post
---

Git commands I mostly use (some might have been forgotten).

Hope I don't have to mention 

    git log
    git status

Add 

    git add -p  #select hunks
    git add --all
    git add -u  #update tracked

Commit, will add changed files and commit:

    git commit -a -m 'your message here'

Amend:

    git commit --amend -m 'message'
    git commit -a --amend --no-edit

Squish, rewrite ... rebase  interactive mode

    git rebase -i <branch_to_rebase_from>

Mingle with origin

    git remote set-url origin <url_of_the_repo>
