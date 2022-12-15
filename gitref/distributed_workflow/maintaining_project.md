---
layout: page
title: "Maintaining a Project"
permalink: /gitref/working_on_github
---

[comment]: <> (TODO: Should break out the GitHub stuff to a separate page)

## Contributing to a project on GitHub

### The GitHub Flow

1. Fork the project
2. Create a topic branch from master
3. Make some commits to improve the project
4. Push this branch to your GitHub Project
5. Open a Pull Request on GitHub
6. Discuss, and optionally continue committing.
7. The project owner merges or closes the Pull Request
8. Sync the updated master back to your fork

This is essentially the Git [Integration Manager workflow]({% link gitref/distributed_workflow/distributed_workflow.md %}#integration-manager-workflow) but using GitHub tooling instead of email.

### Creating a Pull Request

1. Clone the fork of the you want to contribute to locally.
2. Create a descriptive topic branch.
3. Make your changes to the code.
4. Check that the change is good.
5. Commit the change to the topic branch.
6. Push your new topic branch to the GitHub fork.
7. GitHub will notice that you pushed a new topic branch up and present you with a big green button to check out your changes and open a Pull Request to the original project.
    * You can also navigate to the branch in your project and find the option to open a pull request at the url below.
        
        `https://github.com/<user>/<project>/branches`

8. Once you click the big green button you are asked to give the pull request a title and description.
9. When you create the pull request the owner of the project you forked will get a notification that someone is suggesting a change.  The notification will have a link to your pull request.

### Iterating on a Pull Request

Once you have a Pull Request created the project owner can accept, reject, or comment on the pull request.

The project owner can make comments on the diff of the pull request.  Once the maintainer makes their comment, the person who opened the pull request (and anyone watching the history) will get a notification.

Now you just need to commit to your topic branch and push.  This will automatically update the Pull Request.  Adding commits to an existing Pull Request doesn't trigger a notification.  You can communicate and trigger a notification via comments in the pull request.

It is important to note that you can also open a Pull Request between two branches in the same repository.  If you have write access you can push a topic branch to the repository and open a Pull Request on it to the master branch fo that same project to initiate a code review and discuss process.

### Keeping up with Upstream

If your Pull Request becomes out of date or otherwise doesn't merge cleanly you will want to fix it so that the maintainer can easily merge it.  GitHub will test for this, and let you know at the bottom of each pull request if the merge is trivial or not.

You have two main option to fix a non clean pull request merge.

* You can rebase your branch on top of whatever the target branch is (normally the master branch of the repository you forked).  Rebasing will get you a cleaner history, but its more error prone thus the option below is chosen by most developers on GitHub.
* You can merge teh target branch into your branch.

If you want to merge the target branch into your Pull Request to make it mergeable, you would take the following steps.

1. Add the original repository as a remote named upstream.
2. Fetch the newest work from that remote.
3. Merge the main branch of that repository into your topic branch.
4. Fix any conflicts that occur.
5. Push back up to the same topic branch.  Once you do this the Pull Request will be automatically updated and re-checked to see if it merges cleanly.

### Collaborating on Pull Requests

#### References in comments and descriptions

All issues and Pull Requests on GitHub are assigned a uniq number within the project.  If you want to reference any pull request or issue you can simply put `#<num>` in the comment or description.  You can also be more specific.  If the issue or Pull Request lives somewhere else you can write `username#<num> if you are referring to an issue or Pull Request in a fork of the repository you are working on.  In addition to issue numbers you can also reference specific commits by their SHA-1.

Below is an example

[comment]: <> (TODO: Come up with a better example than this random stuff from the book.)

> This PR replaces #2 as a rebased branch.
>
> You should also see tonychacon#1 and of course sachacon/kidgloves#2.
>
> Though nothing compares to https://github.com/schacon/kidgloves/issues/1

#### Github Flavored Markdown

GitHub markdown is supported by nearly every textbox you will find on GitHub

##### Task Lists

Task Lists is a GitHub markdown specific feature, and it allows you to create a list of checkboxes that can be used as lists of thing that need to be done for example in a pull request.  When you mark boxes as checked in GitHub it will correctly update the markdown.

Below is the syntax for Task Lists

```
- [X] Write the code
- [ ] Write all the tests
- [ ] Document the code
```

Which will look like

- [X] Write the code
- [ ] Write all the tests
- [ ] Document the code

##### Code Snippets

You can add code snippets to comments.  This is useful if you want to provide a suggestion within the comments.  Use triple back quotes to denote a code block.
```

```java
for(int i=0 ; i < 5 ; i++)
{
	System.out.println("i is: " + i);
}
```

***Note:*** Due to page formatting the close triple backquote is not displayed above.

##### Quoting

You can use the > symbol do denote a line being quoted.  This is useful if you are referring to specific content in the repository in your text. 

##### Images

GitHub allows you to drag and drop images and will create the link for you.

### Keeping your GitHub repository up to date

Once you fork a repository your fork exists independently from the original repository.

When the original repository has new commits, Git will inform you with a message similar to the example below, but it will not automatically update your work.

> This branch is 5 commits behind progit:master

You can keep your master of your forked project up to date with the original with the following series of commands.  These commands .  .    

Add the source repository and give it a name.	`git remote add progit https://github.com/progit/progit2.git`

Get a reference on progit's branches, in particular master.	`git fetch progit`

Set your master branch to fetch from progit remote.	`git branch --set-upstream-to=progit/master master`

Define the default push repository to origin.	`git config --local remote.pushDefault origin`

After the set up work done by the above commands you can use the series of commands below to keep your fork up to date.

If you are on another branch return to master	`git checkout master`

Fetch changes from progit and merge changes into master	`git pull`

Push your master branch to origin	`git push`

***Note:*** You will need to be careful not to commit changes to master since that branch effectively belongs tot he upstream repository.

## Maintaining a Project on GitHub

### Special Files

### README file

The README file can be in any format that GitHub will recognize (README.md, README, README.asciidoc).  GitHub will render this file as the home page of the project.

Its a good idea to put the following in this file

* What the project is for
* How to configure and install it
* An example of how to use it or get it running
* The license that the project is under
* How to contribute to it.
* As GitHub renders the file add any images that will help better understanding

### CONTRIBUTING file

If you have this file in your repository, GitHub will display its contents when someone opens a Pull Request.  This is a good place to put instructions.

### GitHub Scripting

#### Services

Under project settings you will find a service and hooks section that can be used to integrate with other services and CIs

#### Hooks

GitHub hooks will post an HTTP payload to a URL that you can specify an any event in GitHub that you choose.  A developer screen is provided that you can use for experimenting.

#### GitHub API

You can use GitHub APIs to do things such as add collaborators or label issues.  There is a library available (Octokit) if you don't want to use curl requests.