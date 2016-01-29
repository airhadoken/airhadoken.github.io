---
title: "Best Practices with Git and Private Data"
author: "Bradley Momberger"
---

Every so often a newish bot writer will ask "Where should I put my bot's private data? It shouldn't be on GitHub, right?"  The suspicion underlying the latter question is correct; developers need a different strategy for releasing/maintaining source code vs. managing config data for deployment.  But knowing what developers _shouldn't_ do in this case does not indicate what they _should_ do.

It's not a simple task to manage private data in any project.  In professional projects, we're usually managing deployment data on a Jenkins server and backing up with a widely deployed Word document.  I don't know about you, but for me Jenkins is the wrong tool for projects I maintain myself or build from sanitized source from other providers.  But I would like to keep the data slightly redundantly, allowing for maybe a local and remote copy of data that can easily be restored from either source.

In my Google Apps Script projects, private data management is fairly easy since scripts are self-hosting (scripts are written in the Web-based script editor and stored in Google Drive, so there's no second step for deploying source).  I write a complete script that includes my private keys and tokens, and if I want to publish a script for general consumption, I create a new project with the same script, only sanitized of any private data.

For more complex bot scripts that might require deployment to Heroku or another script engine, things are more complicated.  With Git as the primary means of publishing scripts for general consumption _and_ as the primary means of deploying code to Heroku, it seems to be a natural fit to base your isolation on Git branching strategies.  However, Heroku recommends configuring environment variables.  I'm going to spend the rest of this post talking about why I like the former over the latter, but also going over in depth how to use each strategy for managing your keys and passwords.

# 1 Environment Variable Strategy

The method endorsed by Heroku for setting private data is to set it into the "process environment" of your deployment. All computer operating systems have some form of process environment, which allows certain keyed values to be set before starting a process (i.e. running a program).  Each instance of a running program (a "process") has its own environment, and usually this is a combination of values inherited from the process that launched it (its "parent") and those values set specifically for it.  So, for example, a simple process like the one below adds `FOO` to the environment of the `bash` process started at the command line, and its value can then be echoed.  `LANG` is already set in my shell environment, and it is also inherited to the child process's environment.  The commands `export` and `unset` are also used to manipulate the environment of the shell session (also a running `bash` process).

```console
$ # Set a var for the current shell process instead of a subprocess
$ # Important note: don't put any spaces around the = sign below.
$ export FOO="baz"
$ echo $FOO
baz
$ # Subprocess gets new value for $FOO instead of existing one
$ FOO="bar" bash -c 'echo $FOO $LANG'
bar en_US.UTF-8
$ # Different case; $FOO interpreted to current process's value
$ #  instead of subprocess's value.
$ FOO="bar" echo $FOO $LANG
baz en_US.UTF-8
$ # Remove shell value
$ unset FOO
$ echo $FOO

$
```

[Heroku's set of commands for config management](https://devcenter.heroku.com/articles/config-vars) is very simple: `config:set`, `config:get`, and `config:unset`.  The `set` case uses the same variable-setting syntax as the `bash` examples above, i.e. `name=value` with no spaces around the equal sign.  `config:unset` takes one or more names of variables to unset, just like `unset` does above.  Keep in mind, the environment for Heroku is based on the defaults set up in the buildpack and in Heroku's execution environment, and isn't related to the environment on you local dev machine.  To use the extra environment you've created locally, you need to use [`heroku local`](https://devcenter.heroku.com/articles/heroku-local) to run your script.

## 1.1 Drawbacks of Environment Variables

One major drawback of publishing code with this configuration strategy is that it reduces portability.  True, all operating systems support environment variables, but the method by which they are set is different for Heroku, OpenShift, NodeJitsu, Linode, and so on.  Either your README has to cover each case ("Do `heroku config:set` or `rhc env set` or `jisu env set` or..."), or you have to trust that your users can adapt the configuration process to their own deployments easily.  Depending on the intended users, this may or may not be true.  Your ability to create, say, an installation shell script is limited to just to known supported platforms, or requires a command generation step like `automake` does for your C compiler.  Making execution run with environment variables is its own command as well, as in the `heroku local` reference above; other platforms have their own commands.

Once again, for larger projects this may be desirable, but in the case of something on the scope of a Twitter bot, I recommend a Git branching strategy instead.

# 2 Branch Strategy

The branching strategy relies on how distributed version control systems (DVCSes) operate.  I'll be using Git as my DVCS of reference, because it's what GitHub uses for storage and most cloud app services use for deployment.  The philosophy here is to use version control for config because it preserves history, is more portable, centralizes configuration docs, and reduces the chance of unthinking "oops" moments.

## 2.1 Git-based Deployment

Before getting started, I want to show how a branch strategy is set up at the very core.  First we need two remotes, one to deliver public source code, and a private one for deployment (like Heroku provides). Here's the remotes from [@neoplastibot](https://twitter.com/neoplastibot),

```console
heroku  https://git.heroku.com/neoplastibot.git (fetch)
heroku  https://git.heroku.com/neoplastibot.git (push)
origin  git@github.com:airhadoken/neoplastibot.git (fetch)
origin  git@github.com:airhadoken/neoplastibot.git (push)
```

What we want is a local branch that tracks each of the local branches.  When you do `git clone git@github.com:airhadoken/neoplastibot.git`, the master branch tracking the GitHub master is set up for you.  When you need to push to Heroku for the first time, you can and should create a branch called `private/master` that copies master, and then push to Heroku's instance with `git push --set-upstream`.  This will make your private master branch track Heroku.

Now let's look at our branches.

```console
$ git branch -vv
master          3a5d6e1 [origin/master] Misspelled bot name in readme
private/master  1497833 [heroku/master] PRIVATE config.json vars for mongodb stats connection
```

This is right.  Private goes to heroku, public goes to github.  The private branch has the config vars on top, because it's always put on top of whatever the current public master is. We'll cover this process later on.

# 3 Setup

## 3.1 Environment Strategy

The environment strategy is straightforward but you will need to do it for every deployment, and any other users of your software will also have to go through it.

### Step 1

Set each environment variable you want to use with `heroku config:set`, as you write them

```console
$ heroku config:set INTERVAL_MS=120000
```

Your code, if NodeJS, will have all environment variables in the `process.env` object, i.e.:

```javascript
var interval_ms = process.env.INTERVAL_MS || 120000;
```

### Step 2

Put the same environment variable into a file called `.env` so you can use it with `heroku local`

```
INTERVAL_MS=120000
```

Make sure you add `.env` to `.gitignore` so Git doesn't accientally pick it up to put in a commit you publish to GitHub.

### Step 3

Document the meaning of the environment variable in your README and/or source using the variable.

```
You'll want to add the following environment variables to your Heroku instance before starting:
* INTERVAL_MS: how long to wait between Tweets in milliseconds. Default is 120000
```

## 3.2 Branching Strategy

There are more explicit steps to setting up the branching strategy, but it's worth it for your users.

### Step 0

If you have a Heroku project set up, ensure that you have a remote set up for GitHub and one for Heroku, and a local branch tracking the master branch, as in section 2.1.

### Step 1

Create a file `config.json` at the top of your source tree; it should contain at least an empty object:

```json
{}
```

### Step 2

in your NodeJS script, put a line:

```javascript
var config = require('./config.json')
```
### Step 3

Put a line in your README.md that says:

```
See [config.json](./config.json) for instructions for setting up.
```

The markdown above makes a link to the `config.json` file when published to GitHub.  Try it!

Now we haven't added any variables to the config file yet.  See the next section for that loop.

### Step 4

If you have any config keys you know you want to add, add them now.

Commit `config.json` to the public master branch, then return to the private master for more work

```console
$ git checkout master
Switched to branch 'master'
Your branch is up-to-date with 'origin/master'.
$ git add config.json
$ git add -u
$ git commit
[master <commit id>] <commit message>
 3 files changed, 5 insertions(+)
 create mode 100644 config.json
$ git push
To git@github.com:airhadoken/neoplastibot.git
   <old commit>..<new commit>  master -> master
$ git checkout private/master
Switched to branch 'private/master'
Your branch is up-to-date with 'heroku/master'.
$ git rebase master
```

# 4 Working Live

## 4.1 Environment Strategy

There's nothing more needed to do to manage your environment, but definitely keep a backup of your environment somewhere you'll be able to find it again. Here I'm just going to talk about recovery if you need to ever rebuild your local files or make a new deployment.  Again, this is tailored to Heroku.

### Recover environment for new local files

Really we just need to pull our existing server config in to an .env file to run locally.  This is best accomplished with a simple shell command.

```console
$ heroku config -s > .env
```

There is no output here.  Just check the contents of `.env`

### Recover Heroku environment for new or restarted deployment

If we have the local .env file already, it's enough to make a command line out of it using the `xargs` command

```console
$ xargs heroku config:set < .env
Setting config vars and restarting neoplastibot... done
INTERVAL_MS:  120000
```

## 4.2 Branch Strategy

### Step 1

For each variable you want to add to your script, add it as a JSON key-value pair to `config.json`.  Document it in the same object with (my preference for convention) a string with a key ending in "_NOTES".

```json
{
  "INTERVAL_MS_NOTES": "How long to wait between Tweets in milliseconds.  Default is 120000 if not set here",
  "INTERVAL_MS": 120000
}
```

Don't put private values in your code yet!  Leave them at defaults or empty values at the moment.

### Step 2

Commit your code to the public master branch.  Push to Github.  Then switch to the private branch and rebase your private branch on top of the public one.

```console
$ git checkout master
Switched to branch 'master'
Your branch is up-to-date with 'origin/master'.
$ git add -u
$ git commit
[master <commit id>] <commit message>
 3 files changed, 5 insertions(+)
 create mode 100644 config.json
$ git push
To git@github.com:airhadoken/neoplastibot.git
   <old commit>..<new commit>  master -> master
$ git checkout private/master
Switched to branch 'private/master'
Your branch is up-to-date with 'heroku/master'.
$ git rebase master
```

### Step 3

Add your private data to config.json.  Commit just that private data.

```console
$ git add -u
$ git commit -m "Added config data PRIVATE - DO NOT RELEASE TO GITHUB"
[private/master <commit id>] Added config data PRIVATE - DO NOT RELEASE TO GITHUB
 1 files changed, 1 insertion, 1 deletion
```

The next steps are a loop for working with all code changes you make.


### Step 4

Write code on your private branch, make edits, and run edited code with private configuration to see if it works. 

When your code is working, perform Step 2 again with your new edits.

### Step 5

Push your rebased private branch with force.

```console
$ git push --force
To git@github.com:airhadoken/neoplastibot.git
   <old commit>..<new commit>  private/master -> master (forced update)
```

In this case, we have to do a force-push because the rebase means that the commit tree is no longer just new commits on top of the old ones.  The private-only 

Steps 4 to 6 will repeat each time you want to work live on your code.  You'll note that you need to break up the commits where a new variable is introduced, to the variable itself and to its private value.

### Alternate Cases

It's possible to run into some snags when going through the flow outlined above.  I've provided a "what to do" for the two most common occurrences.

#### Case 1: Git won't let me checkout the master branch

Sometimes you get an error when trying to checkout the master branch: `Your local changes would be overwritten by checkout.`  This is rare with the flow listed above but it certainly can happen in cases where you made changes to your published source without merging them back to your private source (i.e. documentation).  Git can't change the contents of files you've edited in your working tree without committing, so-called "dirty" files.  So you have to `stash` them, which is like making a commit, but it's a commit that goes to a special stack separate from your main commit tree.

```console
$ git checkout master
error: Your local changes to the following files would be overwritten by checkout:
  app.js
Please, commit your changes or stash them before you can switch branches.
Aborting
$ # on the next line, "save" is implied when calling git stash with no other command
$ git stash
Saved working directory and index state WIP on master: <last commit id> <last commit message>
HEAD is now at <last commit id>
$ git checkout master
Switched to branch 'master'
Your branch is up-to-date with 'origin/master'.
$ git stash pop
On branch master
Your branch is up-to-date with 'origin/master'.

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

  modified:   app.js
```

#### Case 2: Oh no! I accidentally checked in to the private branch first!

This is a pretty common occurrence that even experts do.  First thing is to chill, this isn't a reason to panic.  Second is to go to the master branch and "cherry-pick" your commit from the private branch.

> Note about cherry picking:  when you do `git cherry-pick` to stick a commit onto the head of another branch, the result is not the same commit as the commit from which you started, and it has a different ID.  The reason for this is that commit hashes are computed in part by the hash of the commit's parent, so even two commits with the same patch will have different IDs because they start from different commits with different IDs.  However, the hash of the patch itself is also contained within the commit, allowing Git to figure out which commits are redundant when merging branches together.  This becomes important when we do the rebase in the penultimate step.

This is pretty straightforward:

```console
$ git checkout master
Switched to branch 'master'
Your branch is up-to-date with 'origin/master'.
$ # THIS IS NOT PICKING THE WHOLE BRANCH EVEN THOUGH WE'RE USING THE BRANCH NAME AS THE COMMIT REF
$ git cherry-pick private/master
[master <commit id>] <commit message from private commit>
 1 file changed, 2 insertions(+)
$ git push
```

Now that you have a commit on master with the same patch ID, (remember, not the same commit), it will be ignored when doing the rebasing of all your private commits on top of the public master commit.

```console
$ git checkout private/master
Switched to branch 'master'
Your branch is 1 commit ahead of 'origin/master'.
$ git rebase master
First, rewinding head to replay your work on top of it...
Applying: Added PRIVATE INFO - DO NOT RELEASE TO GITHUB
$
```

and you can force-push to the deployment repository from there.

# 5 Conclusion

The Git strategy for managing config for your small applications seems tedious, and in some sense it is.  But the surety of having a record of your config changes, coupled with a reduced reliance on how particular hosting services manage environments, makes the branching strategy preferable for botmaking and other small projects.