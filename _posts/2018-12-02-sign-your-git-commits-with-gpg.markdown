---
layout: post
title: "How to sign a commit in git with a GPG key"
categories: [git, github, gpg]
---

In git, whenever you want to add a change to a repository, a new [commit](https://git-scm.com/docs/git-commit) is created in the repository history.

Besides the actual changes in the repository, every commit includes several metadata fields,
some of them pointing to both the author and the committer:

- *Author*: the person who wrote the code.
- *Committer*: the person who added the code to the repository.

```
    Author:     juandebravo <juandebravo@gmail.com>
    AuthorDate: Wed Nov 21 22:41:04 2018 +0100
    Commit:     juandebravo <juandebravo@gmail.com>
    CommitDate: Wed Nov 21 22:41:04 2018 +0100
```

99% of the times, both author and committer points to the same person; only in distributed teams working
in a flow that requires both [*git format-patch*](https://git-scm.com/docs/git-format-patch)
and [*git apply*](https://git-scm.com/docs/git-apply) may happen that author and committer
are different (e.g. someone providing a new functionality or bug fix, the author, but not having write
permissions in the repository and therefore another person, the committer, would write the commit
on his/her behalf).

Both fields are configurable locally, and there's no way you can ensure a commit uploaded to
a repository hosted in a SaaS product like Github or Gitlab was authored or committed by the person
defined in *Author* and *Committer* fields... unless you enter **commit signature verification**.

Git provides a mechanism to [sign your work with a GPG key](https://git-scm.com/book/en/v2/Git-Tools-Signing-Your-Work).

Working on my macOS High Sierra, these are the steps I followed to get my commits signed:

#### Install git and gnupg

<!-- https://stackoverflow.com/questions/39494631/gpg-failed-to-sign-the-data-fatal-failed-to-write-commit-object-git-2-10-0 -->
```bash
brew install git gnupg gpg-agent pinentry-mac
echo "pinentry-program /usr/local/bin/pinentry-mac" >> ~/.gnupg/gpg-agent.conf
```

#### Generate a GPG key

```bash
gpg --gen-key
```
*Important*: include the email and user name you'd like to use while signing your commits.

#### Obtain the GPG key id of the key you just generated

You will need this number in the following step.

```bash
gpg --list-secret-keys --keyid-format LONG
```

#### Configure username, email and GPG in git

```bash
git config --global user.name <your-user-name>
git config --global user.email <your-email>
git config --global gpg.program gpg
git config --global commit.gpgsign true
git config --global user.signingkey <your-gpg-key-id>
```

#### Restart GPG agent

```bash
gpgconf --kill gpg-agent
```

## Github
Once you start signing your commits, you can let the world know commits pointing to your username in
[GitHub](https://help.github.com/articles/associating-an-email-with-your-gpg-key/) were indeed
committed by you (or at least by someone with access to your GPG private key!).

First of all, export the public key to ASCII armor format:

```bash
gpg --armor --export GPG <your-gpg-key-id>
```

Second step is uploading the public key using [your GitHub key settings page](https://github.com/settings/keys):

1. Click on _New GPG key_
1. Add the GPG public key ASCII representation
1. Click on _Add GPG key_

Since now on, your commits will be labeled with **[Verified]** in GitHub repository history page

![Sign git commit](/gfx/posts/sign-git-commits/verified-commit.png)
