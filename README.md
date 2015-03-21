# git-cheatsheet

#### identify yourself to git ####
 ```
 # Tell git who you are so that your name appears properly in git commit changelog
 git config --global user.name "MyFirstName MyLastName"
 git config --global user.email myemailaddress@example.com

 # And tell git you want branch matching semantics
 git config --global push.default matching
 ```

#### git topic/story branch workflow ####
git maintainers maintain git with this topic branch workflow https://stuff.mit.edu/afs/sipb/project/git/git-doc/howto/maintain-git.html

#### how to checkout lsweb ####
1. manually upload your ssh public key to your gitlab profile here https://gitlab.com/profile/keys
 ```
 cat ~/.ssh/id_dsa.pub | pbcopy
 ```

2. clone lsweb and checkout LS_Stable_v3.0
 ```
 git clone git@gitlab.com:streambox/lsweb.git
 cd lsweb
 git branch --all # inspect what branches where just cloned (for shits and giggles)
 git checkout --track remotes/origin/LS_Stable_v3.0

 ```

#### workflow for a story branch ####
On laptop:
 ```
 # create a personal/temporary integration branch (potential updates).
 # We will use this integration branch later.
 cd lsweb
 git checkout -B pu.lance origin/LS_Stable_v3.0
 git push --set-upstream gitlab pu.lance

 # fork off stable branch:
 git checkout -B card71 origin/LS_Stable_v3.0
 git push --set-upstream gitlab card71
 ```

On server (ssh t3 for example):
 ```
 cd /c && git fetch && git checkout card71 && git reset --hard origin/card71
 ```

On laptop:
 ```
 # make edits to php
 git commit -am "Keep column widths static when adding serial numbers"
 git push
 ```

On server:
 ```
 cd /c && git pull
 # test changes using browser
 ```

On laptop:
 ```
 # make edits to php
 git commit -am "Keep column widths static when adding serial numbers"
 git push
 ```

On server:
 ```
 cd /c && git fetch && git checkout card71 && git reset --hard origin/card71
 ```

...repeat edit/test cycle until you're reasonably confident card71 is solved.

On laptop:
 ```
 # checkout your personal integration branch and merge your changes
 git checkout pu.lance
 git merge card71
 ```

On server do final check of the integration branch which has the card71 story (and possibly other stories)
 ```
 cd /c && git fetch && git checkout pu.lance && git reset --hard origin/pu.lance
 ```
#### in trello.com, how can you see the card numbers? ####
https://trello.com/c/PkIrgKzd/36-show-card-numbers

A lot of people want this feature.
https://trello.com/c/OvKHeqvC/1003-short-ids-for-cards

Always want to know what the IDs for your Trello cards are? This plugin will display the ID for each card!

This chrome plugin allows you to see the card numbers when looking at the board: https://userstyles.org/styles/61623/trello-card-ids

### how can I squash/condense the commits on my story branch? ###
 ```
 cd lsweb
 git checkout card71
 # move/rebase my card71 commits onto the tip of stable LS_Stable_v3.0
 git rebase LS_Stable_v3.0
 # show me the commits on this card71 branch:
 git log --decorate --pretty=tformat:'%h %d %ar %s' master...card71
 sha1=$(git merge-base LS_Stable_v3.0 card71)
 git reset --soft $sha1
 git commit --amend
 git push --force
 ```

### how can I clone and work on the the BCE prototype UI project? ###
Upload your ssh public key to gitlab (same step as step 1 in [how to checkout lsweb](#how-to-checkout-lsweb))
 ```
 # Work on master branch to simplify workflow since we're not even at
 # version 1.0 yet.  Normally though, working on master branch is BAD,
 # since mutual agreement is master is stable.  

 # When we think we're somewhat stable then we can squash master and
 # start forking topic branches from master.

 git clone git@gitlab.com:streambox/bcewebui.git
 cd bcewebui
 git checkout --track origin/master
 # verify what branch I'm on now
 git branch

 # edit, edit, edit
 git commit -am "Fix stuff"
 git push

 # realize need to fix more stuff...edit, edit...think...edit, edit...oh
 # duh! done
 git commit -am "Fix more stuff"
 git push

 # check for upstream changes from other humans on master branch (my
 # current branch)
 git fetch

 # check what upstream humans changed to see how they affected master
 # branch:
 git diff --color ..origin/master

 # their changes look ok, merge the changes from upstream to my local
 # master
 git merge origin/master
 git push
 ```

#### recomended stuff ####

1. tig shows history really nicely

 ```
 # osx:
 brew install tig
 ```

#### random snippets ####

##### commit modified files only #####
I have modified files that are known to git and some fils that aren't yet in git.  I want to commit only the modified files.
 ```
 # commit modified files only
 git diff --name-only --diff-filter=M | xargs git add
 ```

##### add files to git that git didn't previously know about #####
 ```
 # show me the files that git doesn't know aobut
 git status
 git ls-files -o --directory .
 # verify what branch I'm on, so I don't commit new files to master
 git branch
 # ok, i'm on a test branch and those files look right lets add them to git
 git ls-files -o --directory . | xargs git add
 ```

##### t1 needs repo #####
Do this only once to clone repo to t1
 ```
 ssh t1
 ssh-keygen
 cat ~/.ssh/id_rsa.pub # copy to clipboard over ssh
 # manually copy this to your gitlab profile, same step as step 1 in [how to checkout lsweb](#how-to-checkout-lsweb)
 cd /c
 rm -rf .git
 git init
 git remote add origin git@gitlab.com:streambox/lsweb.git
 git fetch
 # next, be careful here: you're doing a hard reset which will destroy
 # any changes on t1, but that doesn't matter since you always commit
 # all of your changes to gitlab anyway, thats a mutal agreement among
 # team members
 git checkout --track origin/LS_Stable_v3.0
 # or git checkout --track origin/lance_tools
 ```

Now that t1:/c/.git exists...

##### I just borked t1 how can I bisect to find the commit that broke t1? #####
Use git bisect
 ```
 ssh t1
 cd /c
 git checkout lance_tools
 git reset --hard origin/lance_tools
 # test with web browser, something is broken, page doesn't load, ok bisect to find the version that broke it
 git bisect start
 git bisect bad
 git checkout lance_tools~100 # I think it broke somewhere between most recent commit and roughly 100 commits back
 # test with web browser, nope, still broken
 git checkout lance_tools~200 # ok, maybe it broke 200 commits ago
 # test with web browser, yes, now it works, ok

 # tell git this version works and git will start bisecting between lance_tools HEAD and lance_tools~200
 git bisect good

 # test with web browser, yes this version is ok
 git bisect good

 # observe git bisects again
 # test with web browser, ok, this version is broken
 git bisect bad

 # repeat  git bisect good, manually test, git bisect bad, manually test, ...

 # when you're done, then reset
 git bisect reset
 ```

