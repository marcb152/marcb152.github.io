---

title: Git quick start 
description: Reminder guide to properly setup git with SSH each time I use a new computer
tags: [Git]
keyword:
    - linux
    - windows
    - git
    - dev
slug: git-quick-start

---

# Notes 🗒️

This markdown is a personal reminder to help me remember how to correctly setup my git repositories using command line and SSH connection/signing.
This is not a full step by step guide designed for everyone, as it assumes you have already generated and added your SSH key to your GitHub account.
I will keep this guide up to date.

# First setup

1. Setup username `git config --global user.name "User"`
2. Setup email `git config --global user.email "me@example.com"`
3. Setup default editor `git config --global core.editor nvim`
4. Configure default branch name to be 'main' instead of 'master' `git config --global init.defaultBranch main`
5. Configure Git to use SSH to sign commits and tags `git config --global gpg.format ssh`
6. Set up the SSH key you would like to use `git config --global user.signingkey /PATH/TO/.SSH/KEY.PUB`
7. Configure commit signature `git config --global commit.gpgsign true`

[See git docs](https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup)

# Initializing repo 🚀

1. Open git bash and locate your project directory `cd PATH/TO/YOUR/PROJECT`
2. Init your git project `git init`
3. Add your .gitignore template to your project, you can find good ones [here](https://github.com/github/gitignore)
4. Check the status with `git status`
5. Add all untracked files `git add ./` OR `git add *`
6. Initial commit `git commit -a -m "Initial commit"`

# Setting up remote 🌍

1. Create a new empty repo on [GitHub.com](https://github.com/new)
    - **DO NOT add a licence, or readme or whatever**, we will do it afterwards, no need to create merge conflicts now.
2. Set up remote `git remote add origin ssh://git@ssh.github.com/USERNAME/PROJECT_NAME.git`
3. `git branch -M main`
4. `git push -u origin main`
5. You can now check if your commit was verified correctly on GitHub!

> [!IMPORTANT]
> You encountered an error? Go check out [GitHub Troubleshooting SSH guide](https://docs.github.com/en/authentication/troubleshooting-ssh)

> [!INFO]
> To make sure everything gets pushed to your remote:
> `git push -u origin --all`
> `git push -u origin --tags`

# Adding README and licence files

## Licence 📜

1. Visit https://choosealicense.com
2. [Add it to your existing repo](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/adding-a-license-to-a-repository#including-an-open-source-license-in-your-repository)

## README 📑

1. Read https://www.makeareadme.com
2. Create your readme easily on https://readme.so/editor
3. Import it to your project, then commit & push

*I am not a readme expert.*

## Other files 📝

- A changelog is a great way to track updates of various releases https://keepachangelog.com
- CONTRIBUTING.md is essential for open-source projects, [see GitHub docs](https://docs.github.com/en/communities/setting-up-your-project-for-healthy-contributions/setting-guidelines-for-repository-contributors)
- You can also add :
  - [An Issue template](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/manually-creating-a-single-issue-template-for-your-repository)
  - [A Pull Request template](https://docs.github.com/en/communities/using-templates-to-encourage-useful-issues-and-pull-requests/creating-a-pull-request-template-for-your-repository)

# Useful links 🔗

> [!TIP]
> You can check whether your SSH GitHub connexion works using `ssh -T git@github.com`.
> It should greet you with `Hi USERNAME! You've successfully authenticated, but GitHub does not provide shell access.`

- [How to tell git about your signing key - GitHub docs](https://docs.github.com/en/authentication/managing-commit-signature-verification/telling-git-about-your-signing-key#telling-git-about-your-ssh-key)
- [Signing commits - GitHub docs](https://docs.github.com/en/authentication/managing-commit-signature-verification/signing-commits)
