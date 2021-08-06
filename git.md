
### GETTING FATAL: REMOTE ORIGIN ALREADY EXIST
	`$ git remote set-url origin "remote url"`
	
### TO SEE WHAT WE ADDED BY LISTING ALL REMOTES. LISTS OUT ALL OF THE REMOTES ATTACHED TO YOUR REPO LOCALLY
	$ git remote -v
	
	$ git config --global user.name "Your Name"
    $ git config --global user.email you@example.com

### AFTER DOING THIS, YOU MAY FIX THE IDENTITY USED FOR THIS COMMIT WITH:
    $ git commit --amend --reset-author

### TO STASH 
	$ git stash
	$ git stash save nameofstash

### SHOW CONTENT OF STASHED FILES WITHOUT APPLYING
	$ git stash list
	$ git stash show -p
	
### TO UNSTASH
	$ git stash apply
	$ git stash apply stash@{0}
	


## GIT FLOW

### Let’s double check that our master branch is up-to-date using *git fetch & git pull*:

	$ git fetch --all
	$ git pull origin master

### ALWAYS FIRST CHECKOUT TO BRANCH 'DEVELOP' BEFORE CREATING THE FEATURE BRANCH	

- TO CREATE A FEATURE BRANCH
	`$ git checkout -b "develop"`
	`$ git flow init`

	> keep entering till it ask for name of branch

- PUSH TO THE FEATURE BRANCH	
	`$ git push origin feature/name`


### TO SEE THE LIST OF CONFIGURATIONS
	$ git config --global --list

### TO OPEN AND EDIT CONFIG FILE
	$ git config --global -e
	
### MERGE/DIFF TOOL
	$ git difftool to use meld for difftool
	$ git mergetool to use meld for merge conflicts
	 

> HEAD MEANS YOUR LOCAL CHANGES ->
UPSTREAM IS THE FILE ON THE MASTERS REPO (i.e. the remote repository that lives on Github)  <-
ORIGIN IS A REFERENCE OF THE REMOTE REPO WE CLONED FROM
MASTER IS THE DEFAULT BRANCH WE HAVE IN OUR REPO
THE -U OPTION TELLS GIT TO CREATE THE BRANCH MASTER ON ORIGIN IF IT DOESN’T EXIST, WHICH IS ONLY REQUIRED THE FIRST TIME YOU PUSH UP CODE TO A BRANCH.

### YOU CAN COMBINE A COMMIT AND ADD TOGETHER (BUT THIS ONLY WORKS FOR ONE TRACKED FILE)
	$ git commit -am "message"

### TO OPEN THE GIT CONFIG FILE (notepad++)
	$ npp ~/.gitconfig
	
### TO RENAME A FILE ON THE LOCAL BEFORE STAGING THE EDIT
	$ git mv fileName NewName

### YOU CAN ALSO USE THE SAME COMMAND TO MOVE FILES
	$ git mv fileName DestinationName

> IF YOU MOVED A FILE LOCALLY OR CHANGED IT'S NAME. GIT WOULD THINK YOU DELETED THE FILE. YOU CAN ALWAYS REALIGN GIT (It adds and update any changes to the working directory, including renames and deletions)
	`$ git add -A`
	
1.  RESETTING FILES
YOU’RE MODIFYING YOUR CODE WHEN YOU SUDDENLY REALIZE THAT THE CHANGES YOU MADE ARE NOT GREAT, AND YOU’D LIKE TO RESET THEM. RATHER THAN CLICKING UNDO ON EVERYTHING YOU EDITED, YOU CAN RESET YOUR FILES TO THE HEAD OF THE BRANCH:
	`$ git reset --hard HEAD`

>  If you have staged a file say example.py ready for commit but you wish to unstage it, which command will you use
  `git reset HEAD example.py`

- OR IF YOU WANT TO RESET A SINGLE FILE:
	`$ git checkout HEAD -- file.js`

- NOW, IF YOU ALREADY COMMITTED YOUR CHANGES, BUT STILL WANT TO REVERT BACK, YOU CAN USE:
`$ git reset --soft HEAD~1`
	
### DELETING A FILE USING GIT. !REMEMBER TO COMMIT AFTERWARDS
	$ git rm file.js

### TO FORCEFULLY REMOVE THE FILE (-R(recursively) WALKS THROUGH THE FILE OR FOLDER, WHILE F REMOVES IT FORCEFULLY)
	$ git rm -rf file.js
	
### IF YOU CHANGE YOUR MIND AFTER REMOVING YOUR FILE, JUST BEFORE YOU COMMIT. IF YOU WANT YOUR FILE BACK. YOU FIRST UNSTAGE IT BY TYPING
	$ git reset HEAD file.js
	> then restore the file with
	$ git checkout -- file.js

### If you wish to overwrite the previous git commit with a new one
`git commit --amend`
	
## GIT HISTORY
1. SHOW LOG OF COMMIT HISTORY, FROM THE LATEST TO OLDEST 
	`$ git log `
	`git log --pretty=oneline`

- Gives the last 2 git entries
    `git log -2`

2. DISPLAY THE LOG IN ONE LINE, IN A SHORT FORMAT
	`git log --oneline --decorate --graph --all`
	`$ git log --all --online -graph --decorate `(this has been aliased tho)
	`git log --oneline --decorate --graph (for only the present branch)`
- YOU COULD SPECIFY A RANGE OF COMMIT ID
	`$ git log 33f23q3...a6se423`
- USING TIME BASED LOG
	`$ git log --since="2 days ago"`
- TO CHECK THE HISTORY ON A PARTICULAR FILE
	`$ git log -- file.js`
- TO SEE THE HISTORY OF WHAT CHANGES HAPPENED
	`$ git show`
	
	
### To update git version on Windows
	git update-git-for-windows
	

```
…or create a new repository on the command line
echo "# Lemp_Stack" >> README.md
git init
git add README.md
git commit -m "first commit"
git branch -M master
git remote add origin https://github.com/Arafly/Lemp_Stack.git
git push -u origin master

…or push an existing repository from the command line
git remote add origin https://github.com/Arafly/Lemp_Stack.git
git branch -M master
git push -u origin master
```
	
## FIXES

1. fatal: refusing to merge unrelated histories
	`git pull origin branchname --allow-unrelated-histories`


```All you need to do is to checkout to the develop branch on your local computer using:

	git checkout -b develop
if you've already created a develop branch for the project on your local computer, then run:

	git checkout develop
```

2. To check the branch you're on:
	`git branch`
	`git branch -vv`
	`git branch --list`
	
3. To switch
	`git checkout branchname`

3. To rename branch br1 to br2
    `git branch -m br1 br2`

4. To undo a branch merge
    `git merge --abort`
- To confirm a merge:
	`git branch --merged`

5. You are working on a project with a colleague. She created a feature branch (feature/user-profile) and pushed it to the repository. This command can you run to pull and create that remote branch locally

    `git checkout -b feature/user-profile origin feature/user-profile`

6.  To delete a branch
    `git branch -d branchname`

7. To add a tag, we will run the following command.
	`git tag -a v1.0 -m "Added first tag"`

8. We can then see the details of this tag by typing:
	`git show v1.0`

9. Push this up to Github.
	`git push --tags origin master`
	

>The HEAD pointer will always be pointing at something, and whatever it is pointing at is the snapshot that you are currently working in. In this case, we see that HEAD is pointing at master which is pointing at the most recent commit. Yes, we created the develop branch just a moment ago, but HEAD is still pointing at master, which means that any changes that we stage and commit will be on the master branch