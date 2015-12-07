---
title: Useful git commands 
layout: post
---

Git commands I mostly use (some might have been forgotten).

Hope I don't have to mention 

    git log
    git status

add

    git add -p  #select hunks
    git add --all
    git add -u  #update tracked

commit, will add changed files and commit:

    git commit -a -m 'your message here'

amend:

    git commit --amend -m 'message'
    git commit -a --amend --no-edit
    
merge, mine! Disregard what others think the changes should look like... 
This is a bad practice, sever mistakes leading here were already made.

    git merge -X ours other_fork/other_branch

Squish, rewrite ... rebase interactive mode

    git rebase -i <branch_to_rebase_from>

Mingle with origin

    git remote set-url origin <url_of_the_repo>

Push to another branch.

    git push origin some_branch:some_other_branch

Don't be scared of the next one!
Force push (may replace what you have with what you may not want to have).

    git push -f

Just switch branches often enough.
Make sure that you know what you are dropping.
After that, regret nothing.

Nice logs, add \[alias\] to `.gitconfig`:

    lg = log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit

Links:


[Atlassian](https://www.atlassian.com/git/tutorials/setting-up-a-repository/git-config)

[ProGit](http://git-scm.com/book/en/v2/Getting-Started-About-Version-Control)