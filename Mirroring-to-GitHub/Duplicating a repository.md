## Duplicating a repository

To create a duplicate of a repository without forking, you need to run a special clone command against the original repository and mirror-push to the new one.

In the following cases, the repository you're trying to push to--like `exampleuser/new-repository` or `exampleuser/mirrored`--should already exist on GitHub. See "[Creating a new repository](https://help.github.com/articles/creating-a-new-repository)" for more information.

### Mirroring a repository

To make an exact duplicate, you need to perform both a bare-clone and a mirror-push.

Open up the command line, and type these commands:

``` bash
git clone --bare https://github.com/exampleuser/old-repository.git
# Make a bare clone of the repository
  
cd old-repository.git
git push --mirror https://github.com/exampleuser/new-repository.git
# Mirror-push to the new repository

cd ..
rm -rf old-repository.git
# Remove our temporary local repository
```

If you want to mirror a repository in another location, including getting updates from the original, you can clone a mirror and periodically push the changes.

``` bash
git clone --mirror https://github.com/exampleuser/repository-to-mirror.git
# Make a bare mirrored clone of the repository
  
cd repository-to-mirror.git
git remote set-url --push origin https://github.com/exampleuser/mirrored
# Set the push location to your mirror
```

As with a bare clone, a mirrored clone includes all remote branches and tags, but all local references will be overwritten each time you fetch, so it will always be the same as the original repository. Setting the URL for pushes simplifies pushing to your mirror. To update your mirror, fetch updates and push, which could be automated by running a cron job.

``` bash
git fetch -p origin
git push --mirror
```

source: https://help.github.com/articles/duplicating-a-repository/
