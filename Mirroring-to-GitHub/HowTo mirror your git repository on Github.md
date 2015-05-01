# HowTo mirror your git repository on Github

## Scope

This HowTo explains how to mirror a git repository from your ChiliProject server to GitHub. Two-way mirroring is possible, both way will be handled in different chapters so that people who only need one way can ignore the other.

The "push" direction (mirror changes from your ChiliProject server to GitHub) is done in a (near) synchronous way in a post-receive hook in your repository on your ChiliProject server. The "pull" way (getting changes from GitHub to your ChiliProject server) is realized with a cronjob and thus is asynchronous. Please note that this means the repositories can be divergent for short periods of time between each run of the cronjob if you work primarily on GitHub repository, this doesn't happen when working primarily on your ChiliProject repository as changes pushed to your ChiliProject repository get immediately pushed to GitHub too.

Ways to notify your ChiliProject installation of changes in your ChiliProject repository can be combined with the syncing methods described herein but are not part of this how-to.

## Prerequisites

*   A GitHub account,
*   A server hosting your ChiliProject installation and the git repository you want to mirror to/from GitHub,
*   The repository on your server should be bare,
*   Basic knowledge of how to create an ssh key.

## Authenticate your server to GitHub

This step is only needed if you need to push from your ChiliProject repository to your GitHub repository or if you want to pull from a private GitHub repository.

Depending on your security needs and/or the size of your admins team, you can opt to either create one ssh key for your ChiliProject server and upload it to your account, thus your server will have the same permissions on GitHub repositories as you do, or you can create use [deploy keys](http://help.github.com/deploy-keys/) on GitHub, i.e. create one ssh key for each repository you need to access on GitHub, each key giving access to one repository only.

Either way, the user on your ChiliProject server that will run the sync to/with GitHub will need a `~/.ssh` directory to store the ssh keys and configs. If your user is called `www-data` and doesn't have the permission to create directories in his home directory, you can create it (as root):

<pre>mkdir ~www-data/.ssh
chmod 700 ~www-data/.ssh
chown www-data ~www-data/.ssh</pre>

### Create a single ssh key

Log in to your server or change to the user that will perform the sync to GitHub. For example, if this user is www-data, run (as root):

<pre>su - www-data</pre>

Append `-s /bin/bash` to this command if your system complains about a missing login shell or the account currently not being available.

Create an ssh key for that user by running the following command:

<pre>ssh-keygen</pre>

When asked for a path to save the key to, just keep the default and press enter, leave the password blank (press enter 2 more times).

Next, install the public key to your GitHub account. The previous command outputs the path to the public key (look for a line like "Your public key has been saved in /path/to/id_rsa.pub"). Copy the content of this file to the [SSH Public Keys](https://github.com/account/ssh) section of your GitHub Account. Please also consult bullet 4\. "Add your SSH key to GitHub" of the [Set up git help page from GitHub](http://help.github.com/linux-set-up-git/#_set_up_ssh_keys).

### Create one ssh key per repository

This step is only needed if you need per-repository ssh keys instead of one ssh key for all repositories.

GitHub provides a way to get authenticated access to a repository without the need to create an extra GitHub account called [deploy keys](http://help.github.com/deploy-keys/), which we can leverage to create ssh keys for individual repositories. I won't go into much detail about the why of the different steps because you should know your way around ssh and linux good enough if you need this level of security. It is assumed the user that will run the sync is `www-data` and the name of the repository to be synced is `some-repo`

Log in to your server or change to the user that will perform the sync to GitHub:

<pre>su - www-data</pre>

Create a named ssh key for the repository for that user:

<pre>ssh-keygen -f ~/.ssh/some_repo</pre>

Leave the password blank.

Add the generated public key to the deploy keys of the target repository according to the [GitHub help](http://help.github.com/deploy-keys/#how_do_i_add_my_deploy_key).

Add the following lines to the `~/.ssh/config` of the user performing the sync:

<pre>Host some-repo-github.com
  IdentityFile /path/to/www-data-home/.ssh/some-repo
  HostName github.com
  User git</pre>

## Add GitHub as a remote

The URL for the GitHub remote will be of the form:

*   if you don't need to authenticate: `https://github.com/thegcat/some_repo.git` (that's the address you find in the `http` section of the addresses GitHub shows you)
*   if you authenticate with only one key: `git@github.com:thegcat/some_repo.git` (that's the address you find in the `ssh` section of the addresses GitHub shows you)
*   if you authenticate with per-repository keys: `some-repo-github.com:thegcat/some_repo.git` (that's the address you find in the `ssh` section of the addresses GitHub shows you with the user removed and the host replaced with the one define in the user's ssh config)

Go into the repository and add the remote as a mirror:

<pre>git remote add --mirror github the-url</pre>

Please be aware that there are known problems with pull requests appearing as commits in mirrors of public GitHub repositories. GitHub is aware of this but doesn't consider it a big enough problem to fix it for public repos (private repos have a work-around to avoid that), we're aware of that too, and I'll update this how-to as soon as we have a confirmed solution to this problem. Also see [#527](https://www.chiliproject.org/issues/527 "pulling garbage revisions from github (Closed)") for more information.

## Pull from GitHub if necessary

If you were using the GitHub repo in the fisrt place, you should sync your local repo with it:

<pre>git fetch github</pre>

## Set up mirroring jobs

### To GitHub

The easiest way for that is to add a post-receive hook to your local repository: it will automatically push to GitHub each time you push to your repository. If you don't have a post-receive hook yet, copy the sample over:

<pre>cp hooks/post-receive.sample hooks/post-receive</pre>

This file must be executable! Then add the push to github to it:

<pre>git push --quiet github &amp;</pre>

### From GitHub

ChiliProject doesn't have a facility to trigger pulls on Repositories it uses, so this has to be done in a cron. Add the following command to your global crontab, or to the the crontab of the user performing the sync (leave out the username for that), this will fetch new commits from GitHub every half hour (adjust the cron run times as needed):

<pre>*/30 * * * *   www-run  git --git-dir=/path/to/some_repo fetch --quiet github</pre>
