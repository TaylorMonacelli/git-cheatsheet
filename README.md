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
 git checkout --track origin/LS_Stable_v3.0

 ```

#### in trello.com, how can you see the card numbers? ####
https://trello.com/c/PkIrgKzd/36-show-card-numbers

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

##### how can I rebase my bce story onto of masterls branch? #####
This will replace the bce foundation with masterls, so the bce story will sit on top of masterls branch.  It doesn't merge bce story onto masterls branch.
 ```
 git fetch
 [ -z "$(git show-ref refs/heads/masterls)" ] && git checkout -t origin/masterls || git checkout masterls
 git reset --hard origin/masterls
 git checkout bce
 git reset --hard origin/bce
 git rebase masterls
 git push --force
 ```

### I want to fix a production bug, how can I create a topic branch to do hack on in order to fix it? ###
1. make sure there aren't any uncommited changes
2. create topic branch based off origin/masterls, then
3. add an upstream tracking branch so that future pulls/pushes will track with origin/ghostfix

 ```
 git status #verify there aren't any uncommited changes
 git fetch # see Note1 below
 git checkout -t origin/masterls || git checkout masterls # See Note2
 git checkout -b ghostfix masterls
 git push -u origin ghostfix
 ```

Note1: Bring my origin/masterls in sync with remote origin/masterls.  If I didn't do this, then I'd have to git checkout ghostfix && git rebase masterls later.

Note2: '|| git checkout masterls' in case masterls branch already exists

### How to delete a file from a Git repository, but not other users' working copies ###
https://gist.github.com/TaylorMonacelli/87456105737b8dba7d7e
