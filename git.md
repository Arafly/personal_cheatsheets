
### GETTING FATAL: REMOTE ORIGIN ALREADY EXIST
	`$ git remote set-url origin "remote url"`
	
### TO SEE WHAT WE ADDED BY LISTING ALL REMOTES. LISTS OUT ALL OF THE REMOTES ATTACHED TO YOUR REPO LOCALLY
	$ git remote -v
	
	$ git config --global user.name "Your Name"
    $ git config --global user.email you@example.com

### AFTER DOING THIS, YOU MAY FIX THE IDENTITY USED FOR THIS COMMIT WITH:
    $ git commit --amend --reset-author

### TO STASH 
	$ git stash save nameofstash
	
### TO UNSTASH
	$ git stash apply stash@{0}
	
### SHOW CONTENT OF STASHED FILES WITHOUT APPLYING
	$ git stash show -p

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


TO SEE THE LIST OF CONFIGURATIONS
	$ git config --global --list

TO OPEN AND EDIT CONFIG FILE
	$ git config --global -e
	
MERGE/DIFF TOOL
	$ git difftool to use meld for difftool
	$ git mergetool to use meld for merge conflicts
	 

HEAD MEANS YOUR LOCAL CHANGES ->
UPSTREAM IS THE FILE ON THE MASTERS REPO <-
ORIGIN IS A REFERENCE OF THE REMOTE REPO WE CLONED FROM
MASTER IS THE DEFAULT BRANCH WE HAVE IN OUR REPO
THE -U OPTION TELLS GIT TO CREATE THE BRANCH MASTER ON ORIGIN IF IT DOESN’T EXIST, WHICH IS ONLY REQUIRED THE FIRST TIME YOU PUSH UP CODE TO A BRANCH.

YOU CAN COMBINE A COMMIT AND ADD TOGETHER (BUT THIS ONLY WORKS FOR ONE TRACKED FILE)
	$ git commit -am "message"

TO OPEN THE GIT CONFIG FILE
	$ npp ~/.gitconfig
	
TO RENAME A FILE ON THE LOCAL BEFORE STAGING THE EDIT
	$ git mv fileName NewName
YOU CAN ALSO USE THE SAME COMMAND TO MOVE FILES
	$ git mv fileName DestinationName
You can ordinarily move files in Linux with
	mv fileName DestinationName
You can still in the same vein, duplicate a file
	cp originalFile.js duplicateFile.js
	
IF YOU MOVED A FILE LOCALLY OR CHANGED IT'S NAME. GIT WOULD THINK YOU DELETED THE FILE. YOU CAN ALWAYS REALIGN GIT (It adds and update any changes to the working directory, including renames and deletions)
	$ git add -A
	
1.  RESETTING FILES
YOU’RE MODIFYING YOUR CODE WHEN YOU SUDDENLY REALIZE THAT THE CHANGES YOU MADE ARE NOT GREAT, AND YOU’D LIKE TO RESET THEM. RATHER THAN CLICKING UNDO ON EVERYTHING YOU EDITED, YOU CAN RESET YOUR FILES TO THE HEAD OF THE BRANCH:
	$ git reset --hard HEAD

OR IF YOU WANT TO RESET A SINGLE FILE:
	$ git checkout HEAD -- file.js
	
NOW, IF YOU ALREADY COMMITTED YOUR CHANGES, BUT STILL WANT TO REVERT BACK, YOU CAN USE:
	$ git reset --soft HEAD~1
	
DELETING A FILE USING GIT. !REMEMBER TO COMMIT AFTERWARDS
	$ git rm file.js
TO FORCEFULLY REMOVE THE FILE (-R WALKS THROUGH THE FILE OR FOLDER, WHILE F REMOVES IT FORCEFULLY)
	$ git rm -rf file.js
	
IF YOU CHANGE YOUR MIND AFTER REMOVING YOUR FILE, JUST BEFORE YOU COMMIT. IF YOU WANT YOUR FILE BACK. YOU FIRST UNSTAGE IT BY TYPING
	$ git reset HEAD file.js
	then restore the file with
	$ git checkout -- file.js
	
GIT HISTORY
SHOW LOG OF COMMIT HISTORY, FROM THE LATEST TO OLDEST 
	$ git log 
DISPLAY THE LOG IN ONE LINE, IN A SHORT FORMAT
	$ git log --all --online -graph --decorate (this has been aliased tho)
YOU COULD SPECIFY A RANGE OF COMMIT ID
	$ git log 33f23q3...a6se423
USING TIME BASED LOG
	$ git log --since="2 days ago"
TO CHECK THE HISTORY ON A PARTICULAR FILE
	$ git log -- file.js
TO SEE THE HISTORY OF WHAT CHANGES HAPPENED
	$ git show
	
	
To update git version on Windows
	git update-git-for-windows
	

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
	
	
fatal: refusing to merge unrelated histories
	git pull origin branchname --allow-unrelated-histories

Contributing to the documentation project
Contributing this project is as easy as ever. After making the necessary contributions that you desire, open the project in your terminal or powershell.

All you need to do is to checkout to the develop branch on your local computer using:

	git checkout -b develop
if you've already created a develop branch for the project on your local computer, then run:

	git checkout develop
Next, make the changes your want to make and then stage, commit and push to the remote develop branch:

	git add -A
	git commit -m "Add more tutorials"
	git push origin develop
You can then login to BitBucket and create a PR to merge changes from the develop branch to master branch.

To check the branch you're on:
	git branch -vv
	
To switch
	git checkout branchname
	
	
	