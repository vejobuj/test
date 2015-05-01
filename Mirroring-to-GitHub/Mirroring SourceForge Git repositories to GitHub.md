# Mirroring SourceForge Git repositories to GitHub

[SemanticScuttle](http://semanticscuttle.sourceforge.net/), your self-hosted social bookmark manager, uses [SourceForge](http://sourceforge.net/projects/semanticscuttle/) as project hosting space. We're utilizing its [Git](http://gitscm.org/) hosting facilities to coordinate development between several developers. Apart from that, we host file releases there, use the bug tracker and have the [demo installation](http://semanticscuttle.sourceforge.net/demo/) on the SourceForge servers.

[GitHub](https://github.com/cweiske/SemanticScuttle) on the other hand is a verrry popular Git repository hosting service that offers some unique features with strong social components.

To attract more developers and make our development progress more visible, I wanted to mirror the SemanticScuttle [main Git repository](http://semanticscuttle.git.sourceforge.net/git/gitweb.cgi?p=semanticscuttle/sc;a=summary) on GitHub. Other people had the [same idea](http://sourceforge.net/apps/trac/sourceforge/ticket/19065) but unfortunately failed because the SourceForge Git servers don't allow outgoing connections.

A bit of searching led me to a [small bash script](https://gist.github.com/656339) that - instead of a _post-receive hook_ - uses a cronjob on a third server (server-in-the-middle) to do the mirroring. Since I have my own server anyway, this solution was chosen.

## Setup

At first, you need to prepare the server-in-the-middle and equip it with a passwordless SSH key. This key needs to be added to your GitHub account. Using deployment keys does not work since they allow pulling only and are meant for private repos. The alternative is to create a second account on GitHub and add it as collaborator in your project.

The next step is mirroring the SourceForge Git repository on your middle server:

``` sh
git clone --bare --mirror git://semanticscuttle.git.sourceforge.net/gitroot/semanticscuttle/sc SemanticScuttle.git
cd SemanticScuttle.git
git remote add github git@github.com:cweiske/SemanticScuttle.git
git config remote.github.mirror true</pre>
```

## Mirroring

After everything is setup, we're going to write our mirror script:

``` sh
#!/bin/sh
cd /home/cweiske/git/SemanticScuttle.git
git fetch --quiet origin
git push --quiet github</pre>
```

Make it executable, add it to your personal crontab (i.e. every 30 minutes) and you're set. Run it manually once - you need to accept GitHub's remote SSH key.

## Using a git alias

Instead of using a shell script, one can use a git alias to keep the configuration in one place:

<pre>$ git config alias.update-mirror '!git fetch -q origin && git push -q github'</pre>

The _!_ before the commands tells git that the command to follow is a shell command. Now the cronjob:

<pre>10,40 * * * *  cd /home/cweiske/git/SemanticScuttle.git && git update-mirror</pre>
