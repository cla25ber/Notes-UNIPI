#advanced_programming #computer_science

[[AP index]] (3)

<p style="text-align:right;"><span style="color:rgb(0, 176, 80)">03/11/2025</span></p>

The main goal of Git is version control, i.e. a system that records changes to files over time. This is fundamental when collaborating on a project, as if someone introduces an error, it is always possible to revert back to the previous (functioning) version.

A simple system of version control could be simply to create new directories whenever the project is changed. This method is local and also very prone to errors, that is why Centralized Version Control Systems (CVCS) were created. In these systems, the files are kept in a centralized server, where client can request them or change them. The advantage is that there are no more local databases to keep track of, but if the server fails then everything is lost. For this reason, many projects now work in Distributed Version Control Systems (DVCS) like Git, where there is a central server that keeps the files (repositories in Git), but when a client wants to use it, it clones it entirely, so that if the server shuts down, the repository was already distributed in the clients' computer and can be easily restored.

Many control version systems just keeps the changes done to files, so, if you want to reconstruct a certain version, you start from the initial version and apply all the changes up to the version you want. Clearly, this method saves on space, but not on time. Instead, Git saves snapshots of the files (if they some are not changed from one version to the other, then for those only a pointer to the previous version is saved). Being a DVCS, Git performs almost always local actions, meaning that there is no need to connect to a server to modify files; all changes are done to your local version, every previous version can be loaded from your local copy, and, when it is possible again, all changes are uploaded to the central server.

Files in Git can be in 3 different states:
- *Modified*: the file has been changed from the last saved version.
- *Staged*: the file is marked as complete and will go into the snapshot when committing.
- *Committed*: the data is safely stored in your database.

When the data is committed, it is stored in a Git directory (repository). The working directory is on your local machine and contains the latest version of the data, if it is remote it can be pulled from its correspondent Git directory. The files in the working directory can be modified, once one is satisfied with them they can be added to the staging area. When all the changes will have been made they can be committed and will create the new version of the project, which will be saved as a new commit in the repository.

### Git basic commands

As we said, the main actions that can be performed with Git are: creating a repository, cloning an already existing one, pushing changes and committing them.

##### Configuration 

To check on the configuration of your version of Git you can run the command:
```Shell
	git config --list
```
It is also possible to set up your username name and your user email with the following commands:
```Shell
	git config --global user.name "<username>"
	git config --global user.email <email>
```
where \<*username*\> is the username you want to set, and \<*email*\> is the email you want to set.

This information is needed to ensure that every change committed can be traced back by who did it (you will not be able to commit changes without this information).

With specific options, you can use "git config" to have aliases of git command, for example writing "git c" instead of "git commit".

##### Getting a repository

To create a new repository, it is just needed to go into the directory of the project and type:
```Shell
	git init
```
This command will create a directory ".git" inside the project, which will keep all the necessary files for the repository, the so-called repository skeleton. Obviously, at this point no version control is being performed as no file has been added or committed to the repository.

One could use an already existing repository, in that case it can be cloned, the command for that is:
```Shell
	git clone <source> <dir>
```
where the \<*source*\> is the source of the repository (typically a GitHub URL), and \<*dir*\> is the working directory that will be the destination for the project.

When you clone a repository you can get more information about it by running:
```Shell
	git remote -v
```

![[Git Untracked Files.png]]

In the example project, the command "git init" was executed, every file is now marked with an U as "untracked". This means that have not been changed since when the repository was created, but also none was added.

##### Status of files

To check the status of the files of the repository, the following command is used:
```Shell
	git status -s
```
This will show a short description of the files' status; without the -s option it will be more wordy.

Sometimes it is useful to ignore some files inside a directory, meaning that the repository should maintain the version control of the others. To do so, a ".gitignore" file can be created where, through regular expression, the files to be ignored can be listed. For example, by writing \*.o we tell git to ignore every file ending with the "o" extension.

To stage a file the command used is:
```Shell
	git add <filename>
```
where \<*filename*\> is the name of the file that you want to stage (it can be an entire directory).

![[Git Added Files.png]]

In this example, inside the ".gitignore" file, the file "notex.txt" was told to be ignored, so it was greyed out. All the other files were staged, so now they are marked with an A for "added". After getting added the file "index.html" was modified, creating a difference from the staged one, so it is marked with an M as in "modified".

When a file gets modified (marked with an M) it is possible to see where it has been changed and what has been changed by running the following command: 
```Shell
	git diff
```
This will tell the differences between the staged version and the unstaged version. If you want the difference between what you have staged and what is committed, you can run:
```Shell
	git diff --staged
```

Finally, once all the file have been staged, you can commit the changes to the repository. To do so you can run:
```Shell
	git commit -m <message>
```
where \<message\> is the message that describes the changes made. You cannot commit without writing a message, if you omit the "-m \<*message*\>" part, your default text editor will open and will ask you to write a message for the commit.

![[Git Committed Files.png]]

One the files have been committed they will return to their original color (except the ignored ones which will remain greyed out).

It is possible to skip the staging area, committing files directly, by executing:
```Shell
	git commit -a -m <message>
```
which will first stage automatically every modified file, and then will commit them with the message \<*message*\>.

To view the history of commits you can use:
```Shell
	git log
```
which will show in reverse chronological order all the commits and messages performed on the project. Various options can be added to the command to limit the number of commits showed.

##### Undoing things

If you have committed something, but then realize that you missed something or you want a different commit message, you can, first add the files you missed or change the things you forgot to change, and then run:
```Shell
	git commit --amend
```
Instead of creating a new commit, this command will replace the last one, so not to pollute too much the history of commits.

If you have a staged file and modified it, but you want to delete those changes, you can run:
```Shell
	git restore <filename>
```
where \<*filename*\> is the name of the file you want to restore.

If you want to unstage a file you can run:
```Shell
	git restore --staged <filename>
```
which will unstage the file called \<*filename*\>.
##### Tagging

Sometimes it might be important to set some tags in the history of commits to mark important moments, to do so you can execute:
```Shell
	git tag -a <tagname> -m <message>
```
This will create a tag in the current commit with name \<*tagname*\> and message \<*message*\>. If you do not put a message, you will be asked to, like it was for "git commit".

It is also possible to tag previous commit by executing:
```Shell
	git tag -a <tagname> <commitname>
```
where the commit \<*commitname*\> will be tagged with \<*tagname*\>. To find the name of a commit you can use "git log".

Tags are useful to retrieve information about a certain commit, to do so you can run:
```Shell
	git show <tagname>
```
which will give you information about the commit tagged with \<*tagname*\>.
### Git branching

The key feature of Git is its branching mechanism. First of all, we must remember what a commit actually is:
![[Git Creating a Branch.svg]]

A commit is a pointer to a snapshot, i.e. the files as they were on that commit. When a new commit is made, the new one points to its snapshot, but also points to the previous commit. Every commit is done on the master branch (it is called "master" by default, you can change its default name using "git config"). HEAD is a special pointer that points to the branch you are currently in, so, if none are created, it will always point to master. It is possible to create new branches using the following command:
```Shell
	git branch <branchname>
```
where \<*branchname*\> is the name of the branch that you want to create. This command does not switch HEAD; to switch to another branch you can use the command:
```Shell
	git checkout <branchname>
```
where \<*branchname*\> is the name of the branch that you want to move to; this will move the HEAD pointer to that branch.

Running "git branch" without any arguments will give you a list of all the currently active branches.

The command:
```Shell
	git checkout -b <branchname>
```
will create the branch \<*branchname*\> if it does not exist and will also move to it.

If after moving to a new branch a commit is executed, the two branches will now point to different commits. Now, if we switch back to the previous branch, the files on the working directory will change to that of the commit the branch is pointing to, and if we commit some other changes from there we will have created diverging branches.
![[Git Diverging Branches.svg]]
You can run "git log" with the options "--decorate" and "--graph" and "--all" to get more information about all the commits and their structure.

##### Merging

Let us suppose to be in the following situation:
![[Git Merge Example 1.svg]]
One thing we could do is merge the various branches that we have, to do so we can use the command:
```Shell
	git merge <branchname> -m <message>
```
which will merge the current branch (the one with HEAD), to \<*branchname*\> with merge message \<*message*\>. As always, the message is mandatory. In the case of "Master" and "Branch 3" this will be just a "fast-forward", because they reside on the same path. This would just make "Master" move to "Branch 3". Once the two branches point to the same commit one can be deleted, which can be done using the command:
```Shell
	git branch -d <branchname>
```
that will delete the branch \<*branchname*\>.

So, after performing these actions, merging "Master" and "Branch 3" is a completely different situation, as "fast-forward" cannot happen. Instead, a new commit is created, one that unites both "Commit 4", "Commit 5" and their common ancestor, i.e. "Commit 3", obtaining the following:

![[Git Merge Example 2.svg]]

Only the "Master" branch has been moved because it was the one with the HEAD pointer when the merging occurred.

Unfortunately, it is not always this simple, if both "Commit 5" and "Commit 4" have performed changes to the same files on the same lines, what we get is a <span style="color:rgb(255, 0, 0)">merge conflict</span>. The merge procedure will fail and we will be asked to fix the conflicts before merging again. Once they have been solved, the merging procedure will be performed without any problems.

Another way to merge diverging branches is rebasing, in the previous example we could have rebased "Branch 3" into "Master" with the command:
```Shell
	git rebase <branchname>
```
which rebases the current branch into \<*branchname*\>.

The result of rebasing would be the following:

![[Git Rebasing Example.svg]]

Regarding the final "Commit 6", it would be same as the "Commit 6" generated by the "git merge" command; however, the history of commits would be different, as the rebased branch would be eliminated, resulting in a much cleaner and linear history.

Typically, in shared projects it is better to merge, as it preserves history. Generally, if you want a clean history it is better to rebase, otherwise you can merge.

### Remote Git

Git projects are mostly shared remotely, meaning that they are hosted somewhere and you access them. The "remote" is the name that will be given to the remote project many are collaborating on. For example, when you clone a repository, the remote would be where you cloned it from, and it is called by default "origin".
To see the remotes you are working on you can use the command:
```Shell
	git remote -v
```
which will tell you the names of the remotes and also their URLs.

When adding a new remote you can also use the command:
```Shell
	git remote add <remote> <url>
```
which will add the repository saved at \<*url*\> with the name \<*remote*\>.

It is also possible to rename remotes using:
```Shell
	git remote rename <oldname> <newname>
```
which will change the name of the remote called \<*oldname*\> into one called \<*newname*\>.

When you clone or add a remote, it might happens that somebody has added some changes to the remote. To get these new changes and update your working directory you can use:
```Shell
	git fetch <remote>
```
which will download the new changes added to \<*remote*\> and create a new branch that stores the changes and that can be merged afterwards.

Typically, in order to immediately merge the changes into the working directory, the pull command is used: 
```Shell
	git pull <remote>
```
which works just like the fetch command, but also merges the changes from \<*remote*\>.

After having pulled changes from a remote, you might want to push your own, to do so, the command is:
```Shell
	git push <remote> <branch>
```
which allows you to write on the \<*remote*\> the changes from your \<*branch*\>.

Pushing changes is not always that easy, if someone has pushed their changes from the last time you fetched the changes, then your push will be rejected. You will have to integrate the changes pushed before yours before you can actually push yours.

It is possible to force your changes using the "--force" option to the push command, although it is not recommended. 

### GitHub

GitHub is the largest host for Git repositories. You can sign up on GitHub and host your repositories there by associating your SSH key to our account.

To put a project on GitHub you create a new repository on the website, then from the command line you can add that remote, and push your commits.
```Shell
	git remote add <remote> <GitHub_url>
	git branch -M <branchname>
	git push -u <remote> <branchname>
```
You add the \<*GitHub_url*\> as a remote on your machine and you call it \<*remote*\>. Then you rename your branch to \<*branchname*\>. Finally, you push the changes in \<*branchname*\> into \<*remote*\>, which is the remote containing the information of the GitHub repository.

If you have set up your SSH key on your GitHub account, it will be recognized when you push, otherwise it will ask you your Personal Access Token, 

#### References

- Git - <a href=https://git-scm.com/book/en/v2>The Pro Git Book</a>
- Git tutorial - [Git Tutorial for Absolute Beginners](https://www.youtube.com/watch?v=CvUiKWv2-C0)