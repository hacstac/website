---
title: "Step-By-Step Guide To Push Your First Project On GitHub!!"
date: 2019-09-01T20:00:00+08:00
tags: ["Git", "Programming"]
categories: ["Git"]
author: "Akash Rajvanshi"

autoCollapseToc: false
---

### Steps:

> Steps that can make your work half!!

 First select your project & open your terminal in your project’s root directory.

<!--more-->

#### 1. Check for Git Version

```bash
git --version
```

> If it is not showing the version of git then go to the official website of [git](https://git-scm.com/) and download the git according to OS of your system.

#### 2. If we are setting up the git for the first time, we can configure the git with name & email.

```bash
git config --global user.name "Your\Name"
git config --global user.email "Your-Email"
```

#### 3. Initialize Git Repository

```bash
git init
```

> Note:- On your Project’s root directory

It initializes the git repository in local project & will make . git folder that contain important folders and files.

#### 4. Commiting files into the git repo.

There are three steps :-

- Step 1  : We need to add a file to staging area .

```bash
git add <File\Name> `{{For Single File}}`
git add . `{{For all the files in current Directory}}`
```

> Note :- In place of <File\Name> add your file and if you want select all the files in current directory then use `{{ . }}` or `{{ * }}`

Staging area  :- Staging area is that area where we can buy some items and put them in bucket .

To check,if files are added or not,use this:

```bash
git status
```

- Step 2  : Commit a file into the git repo is to write a commit message.

> m = message

```bashs
git commit -m "First Commit"
```

> Note  :- In the “ Double Quotes ”, you should write your message.

- Step 3  : Push the file into a remote repository. `{{Github}}`

Now create your account on git-hub and create a repository .
To add files in your remote repo use this :

```bash
git remote add origin git@github.com:"Username\on\github"/"Repository\Name"
```

> Note  :- This above command is a single command & Now place your git-hub username (without any quotes) and put your repository name (without any quotes).

Like this

```bash
git remote add origin git@github.com:XYZ/project.git
```

And if you don’t understand this then go to your repository on git-hub and click on clone or download button and copy url with SSH method.

To check

```bash
git remote -v
```

#### 5. Create SSH Key.

> Why we use SSH?

- By using the ssh protocol, we can connect and authenticate to remote servers and services. With ssh keys we can connect to GitHub without supplying our username and password at each visit with the help of passphrase.

- In HTTPS method, you will need to fill our username and password at every visit, which will be very inconvenient.

- Git associates a remote URL with a name and our default remote is usually called “ Origin ”

- 1. Generating New ssh key and adding it to a ssh agent.

```bash
ssh-keygen -t rsa -b 4096 -C "email"
```

Note  :- Place your email in “double quotes”.

This creates a new ssh key using the provided email as a label.

1.1. For default file Press `{{ ENTER }}`

```bash
/home/`{{usernameofpc}}`/.ssh/id\rsa: ENTER
```

1.2. Enter a passphrase.

1.3. Now our identification has been saved in

```bash
Private Key : /home/`{{usernameofpc}}`/.ssh/id\rsa
& Public Key : .ssh/id\rsa.pub
```

- 2. Adding our ssh key to the ssh agent

```bash
eval "$(ssh-agent -s)"
it gives like `{{agentid : 15800}}`
```

- 3. Now we add SSH Private key to ssh-agent to our default path.

```bash
ssh-add ~/.ssh/id\rsa
```

- 4. Adding a new ssh key to your github account.

Copy the ssh key to our clipboard.

1. Automatic Method (Using Xclip)

```bash
# On Ubuntu
sudo apt-get install xclip

# on Manjaro
Open Octopi -> Download Xclip

$ Xclip-set clip <~/.ssh/id\rsa.pub
```

2. Manual Method

```
Home/.ssh/id\rsa.pub
Open this file and copy your key.
```


Now goto [github.com](https://github.com/)➢➢ Under Profile Photo (Drop Down) ➢➢ Settings ➢➢ Use SideBar `{{ SSH & GPG Keys }}` ➢➢ Then go to this directory on your computer `{{Home/.ssh/idrsa.pub}}`
- Open this file and copy your  key.
- Create new SSH key ➢➢ Title the filed with descriptive label ➢➢ Paste key into a “Key Field” ➢➢ Click on `{{ Add SSH Key }}`
- Now for Testing SSH Connection.

```bash
ssh -T git@github.com
```

After running this , it will show this messege on terminal!!

```bash
Hi `{{ USERNAME }}`! You've successfully authenticated but github does not provide shell access.
```

#### 6. Final PUSH

```bash
git pull --rebase origin master
git push origin master
```

- Note  : Change ‘master’ whatever branch to push.

Then you can successfully push your file to remote server and you can setup a SSH Connection.

To check the connection

```bash
git log
```

---

### Some other useful git concepts.

> There are concepts that can help you to understand the git more deeply.

#### 1. Create a new branch.

```bash
git checkout -b Branch_name
```

If you want to switch back to master branch.

```bash
git checkout master
```

If you want to delete the branch.

```bash
git branch -d Branch_name
```

#### 2. Update and Merge.

To update local repository to the newest commit

```bash
git pull
```

To merge another branch in active branch.

```bash
git merge <branch>
```

In both cases git tries to auto-merge changes. unfortunately this is not always possible and results are in conflicts.

we can merge those conflicts manually by editing the files shown by git.

```bash
git add <filename>
```

Before merging changes we can preview.

```bash
git diff <source-branch> <target-branch>
```

#### 3. Tagging.

we can give tag (line 1.0.0)

```bash
git tag 1.0.0 `{{1b2eld63ff}}`
```

- Note  : `{{ this is the first 10 characters of commit id. }}`

#### 4. Git Log.

By Git Log we can study repository history.

Advanced  :

1. See only the commits of a certain author.

```bash
git log --author <name>
```

2. Very compressed log.

```bash
git log --pretty=oneline
```

More preferable format

```bash
git log --pretty=format:"%h - %an, %ar : %s"
```

3. It will only show the files that have changed.

```bash
git log --stat
```

#### 5. Replace Local Changes.

```bash
git checkout --filename
```

- (if we did something wrong) then this will replace file with the last content in HEAD.

If we want to drop all local changes and commits, fetch the latest history from server

```bash
git fetch origin
git reset --hard origin/master
```

#### 6. Never forget to create your .gitignore file.

- Gitignore file ➢➢ Is a file that specifying the files or folders that we want to ignore.

There are several ways to specifying those

- Firstly, you can specifying by the specific filename. Here is an example, let’s say we want to ignore a file called readme.txt , then we just need to write readme.txt in the  .gitignore file.
- Secondly, we can also write the name of the extension. For example, we are going to ignore all  .txt files, then write  \*.txt.
- There is also a method to ignore a whole folder. Let’s say we want to ignore folder named test. Then we can just write test/ in the file.

Like this

![gitignore_Example](https://cdn-images-1.medium.com/max/490/1*yqXxPyX3MtgQiqGdcgrc-g.png)

#### Useful Hints

1. Built in git Gui.

```bash
gitk
```

2. Use colorful-git output.

```bash
git config color.ui true
```

3. Use interactive adding.

```bash
git add -i
```

#### Some useful guides, Repositories & Tutorials

- [github/hub](https://github.com/github/hub) `{{ A command-line tool that makes git easier to use with GitHub}}`
- [15 Git Commands You May Not Know](https://zaiste.net/15-git-commands-you-may-not-know/)
- [Adding an existing project to GitHub using the command line](https://help.github.com/en/articles/adding-an-existing-project-to-github-using-the-command-line)
- [How to become a Git expert](https://medium.freecodecamp.org/how-to-become-a-git-expert-e7c38bf54826)
- [Resources to learn Git](http://try.github.io)
- [GitHub Handbook](https://guides.github.com/introduction/git-handbook/)
- [Learn Git Branching](https://learngitbranching.js.org/)
- [jlord](https://github.com/jlord) / [git-it-electron](https://github.com/jlord/git-it-electron)
- [GitHub Learning Lab](http://lab.github.com)
- [Git-Tower](https://www.git-tower.com/windows)
- [Resources To Help You Learn Git](https://speckyboy.com/resources-for-learning-git/) `{{ SpeckyBoy }}`
- [Visual Git Reference](https://marklodato.github.io/visual-git-guide/index-en.html)
- [Complete Resources To Learn Git](https://cssauthor.com/resources-for-learning-git-and-github/) `{{ CssAuthor }}`
- [netroy](https://gist.github.com/netroy) / [git tutorials.md](https://gist.github.com/netroy/1319215)

#### GitIgnore

- [Git- gitIgnore Documentation.](https://git-scm.com/docs/gitignore)
- [.gitignore file — ignoring files in Git | Atlassian Git Tutorial](https://www.atlassian.com/git/tutorials/gitignore)
- [gitignore.io](https://www.gitignore.io/)

#### Tutorials

- [Git Complete: The definitive, step-by-step guide to Git](https://www.udemy.com/course/git-complete/) `{{ Udemy }}`
- [Git a Web Developer Job: Mastering the Modern Workflow](https://www.udemy.com/git-a-web-developer-job-mastering-the-modern-workflow/) `{{ Udemy }}`
- [GitHub Ultimate: Master Git and GitHub — Beginner to Expert](https://www.udemy.com/github-ultimate/) `{{ Udemy }}`
- [Git Going Fast: One Hour Git Crash Course](https://www.udemy.com/git-going-fast/) `{{ Udemy }}`
- [Git & GitHub Bootcamp & Integration with most popular IDEs](https://www.udemy.com/git-bootcamp-with-github-learn-step-by-step/) `{{ Udemy }}`
- [Git Essential Training](https://www.lynda.com/Git-tutorials/Git-Essential-Training/100222-2.html) `{{ Lynda }}`
- [Code School: Git Real](https://www.pluralsight.com/courses/code-school-git-real) `{{ Pluralsight }}`

#### Books

- [Git Pocket Guide](https://www.amazon.in/Git-Pocket-Guide-Richard-Silverman/dp/1449325866)
- [Pro Git](https://github.com/progit/progit2)
- [Git Essentials](https://www.oreilly.com/library/view/git-essentials/9781785287909/)

---
