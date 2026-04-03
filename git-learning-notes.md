# What we need?

    we need the following:
    1. An IDE (like VS Code)
    2. A terminal (in mac it could be WRAP // in windows it could be GITBASH)
    3. For documentation refer to docs.chaicode.com

# GIT is the software whereas GITHUB, GITLAB are different services that we can use online to use the GIT software

# WHY DO WE NEED GIT?

In layman terms, GIT is like a checkpoint that we need to refer to the codes that we have written.

So in technical terms it is a VCS (Version Control System)

# Installation:

STEP 1:
Download and Install Git in the System
STEP 2:
Create an account on GITHUB - This will help in assigning the username when we use GIT in the System

# TERMS AND TERMINOLOGIES:

Repo - it is the same concept of a folder or directory.

check cersion - git --version

# git status - this is to check if there is any such repo in the current directory.

    check with pwd..
    and then run git status

    create a few folders - one, two, three, four..

    it is not necessary to track each and every folder through git. only create repo of the folders you want to track through git.

# git config:

    this is to configure git for the first time in your system with the username and password that you created on github...

        git config --global user.email "your@email.com"
        git config --global user.name "your name"

    you can check the config settings later:
        git config --list

    (optional)
        To change the code editor from VIM to VS Code:
        git config --global core.editor "code --wait"

# git init:

    this is to initialise a folder to make it a git repo..

    for eg.. cd one
    now you are inside one folder..

    now if you do git init
        this folder "one" will be initialised into git repo..

        a hidden file will be created inside one -  .git

        you can check this by ls -la

# Write -> Add -> Commit

    this is the flow that we follow for creating check points.

        git init --> Working dir --> git add --> staging area --> git commit --> Repo --> git push --> Github

# ****\*\*\*\*****to do list to understand the flow:

            in folder one --
                create a file (file1.txt)
                    write something in it -- "this is my file one for git course"
                create another file (file2.txt)
                    write something in it -- "this is my file two for git course"

# now git add :

    common syntax
        git add file1.txt file2.txt
    if you do not want to add one by one, and add all the file all at once;
        git add .

    this will add all the files / selected files to the staging..
    now check again with git status

    if you want you can again unstage these files from here :
        git rm --cached file1.txt

# git commit:

    common syntax :
        git commit -m "message"
            eg.. git commit -m "first commit"


    ************ now lets say you have added more content to file2.txt ... "adding more content to file two"
        now again check with git status
            this will now show that changes not staged for commit

    so again repeat the same things to commit again..
        git add file2.txt
        git commit -m "add content to scnd file"

# git log:

    this is to check all the commits made..
    there are many things that git log tells:

    to shorten what git log prints and to get better checks of errors.. you can use flags..

    --oneline : this will only show oneline with commit id and message

# gitignore:

    there are many such files that we do not want git to keep a track of.. eg.. .env etc..
    so we create a file in the name of .gitignore and then inside the file we name the files to be ignored..  eg.. node_modules, .env

    now git will not track the names mentioned in .gitignore

    so just repeat the add and commit to commit the .gitignore file to your code base...

# .gitkeep:

    git generally does not keep a track of the empty folders..

    this is generally used in cases where there are empty folders but we want git to keep a track of these folders and tell us whether we have commit them or not..

# GIT BRANCHES:

    common syntax: git branch

    Branches in git is a way to work on different projects at the same time.

    Head: Head is the pointer that points to the current branch

suppose you want to create different branch bug-fix

     git branch bug-fix
        but check that you are in master branch before creating any new branch..

    to go on to that branch
        git switch bug-fix

    now lets say if you have a code to fix a bug.. for now lets say fixes.txt

    so you created this file fixes.txt, but remember that you are now in a different branch, not the master one..

    git switch -c dark-mode
        this will not only switch the branch but will also create the new branch with the name "dark-mode"

    git checkout bug-fix
        this is same as git switch command..

# git merge:

    go to the main branch and then merge the other branch into it..

        git checkout master
        git merge bug-fix

        in case of conflicts:
            first resolve the conflict and then add the file once again and then commit it once again...

# branch rename:

    git branch -m <old branch name> <new branch name>

# delete branch:

    git branch -d <branch name>

# git push : to git hub...

    0) Quick check (do you already have an SSH key?)

            Open PowerShell (or VS Code Terminal) and run:

            ls ~/.ssh


            If you see something like id_ed25519 and id_ed25519.pub, you already have keys (skip to step 3).
            If not, do step 1.

    1) Generate SSH key (recommended: ed25519)

            Run:

            ssh-keygen -t ed25519 -C "your_github_email@example.com"


            When it asks:

            Enter file: just press Enter (default path is perfect)

            Passphrase: you can set one (more secure) or press Enter for none

            This creates:

            ~/.ssh/id_ed25519 (private key — keep secret)

            ~/.ssh/id_ed25519.pub (public key — add to GitHub)

    2) Start ssh-agent + add your key

            In PowerShell, run:

            Get-Service ssh-agent | Set-Service -StartupType Automatic
            Start-Service ssh-agent
            ssh-add $env:USERPROFILE\.ssh\id_ed25519


            If it asks for passphrase, enter it.

            ✅ Confirm key added:

            ssh-add -l

    3) Add the public key to GitHub

            Copy the public key:

            cat ~/.ssh/id_ed25519.pub


            Copy the full output (starts with ssh-ed25519).

            Now in GitHub:

            Go to GitHub → Settings

            SSH and GPG keys

            New SSH key

            Title: something like My Windows Laptop

            Paste the key → Add SSH key

    4) Test the SSH connection

                Run:

                ssh -T git@github.com


                First time it may ask:

                Are you sure you want to continue connecting (yes/no)?

                Type:

                yes


                ✅ Success message looks like:

                “Hi <username>! You've successfully authenticated…”

    5) Create a GitHub repo (empty)

                On GitHub:

                Click New repository

                Name it (example: ems-backend)

                Do NOT add README / .gitignore / license (keep it empty) — since you already have commits locally.

    6) Add GitHub as remote using SSH

                Go to your project folder in terminal, and run:

                git remote -v


                If nothing shows, add remote:

                git remote add origin git@github.com:YOUR_USERNAME/YOUR_REPO.git


                Example:

                git remote add origin git@github.com:suvasanyal/ems-backend.git


                ✅ Confirm:

                git remote -v

    7) Push your branch to GitHub

                First check your branch name:

                git branch


                If it shows main, push like:

                git push -u origin main


                If it shows master, push like:

                git push -u origin master


                After this, future pushes are just:

                git push

                VS Code part (very simple)

                After SSH is set:

                Open your project in VS Code

                Source Control (left panel) → Push/Sync

                It should push without asking for password/token.

    Common issues + quick fixes
                ❌ “Permission denied (publickey)”

                Run:

                ssh-add -l


                If it shows no keys, add again:

                ssh-add ~/.ssh/id_ed25519

                ❌ You created the GitHub repo with README and now push errors (non-fast-forward)

                Fix:

                git pull origin main --rebase
                git push

                ❌ Remote is HTTPS but you want SSH

                Check:

                git remote -v


                If it’s https://..., change it:

                git remote set-url origin git@github.com:YOUR_USERNAME/YOUR_REPO.

            ❌ You have pused codes earlier in diff project and now like to push a diff repo:

                    Git likely tried to use a previously saved (but now invalid) credential automatically (Windows Credential Manager / Git Credential Manager). GitHub also doesn’t allow password auth for HTTPS pushes anymore, so if the stored credential is wrong/expired, you get this error immediately.


                ✅ Fix Option A (Recommended): Switch repo to SSH (no token prompts)

                    1. Remove the HTTPS remote and add SSH remote

                        git remote set-url origin git@github.com:Suva87/simple-Nodejs-project-for-docker.git

                        git remote -v

                    2. Test SSH auth to GitHub

                        ssh -T git@github.com

                            If you see: “Hi Suva87! You've successfully authenticated…” ✅

                            If it asks “Are you sure you want to continue connecting” → type yes

                    3. Push

                        git push -u origin main

                ✅ After this, future pushes will be smooth.

                ✅ Fix Option B: Stay on HTTPS but use a Token (PAT)

                    1) Remove the wrong cached credential (so it will prompt)
                        Open:
                            Control Panel → Credential Manager → Windows Credentials

                        Find entries like:
                            git:https://github.com
                            github.com
                        Remove them.

                    2) Push again

                        git push -u origin main

                        use your PAT credentials to authenticate.





