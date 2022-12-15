---
layout: page
title: "Maintaining a Project"
permalink: /gitref/maintaining_a_project
---

[comment]: <> (TODO: Below This was a straight copy paste from notes so need to clean up formatting)

## Distributed Git - maintaining a project

	• You can apply a patch generated with git diff using a command similar to the below…
		○ $ git apply /tmp/patch-ruby-client.patch
		○ git apply is all apply all or abort command.  Either it does it all or nothing at all.
	• Git patch is another command you can use, but git apply is more conservative and makes less assumptions.

	Determining What is Introduced
		
		○ When you have a topic branch that contains contributed work you may want to review all of the commits that are in this branch but that aren't  in your master.  You can do this with the command…
			§ $ git log contrib --not master
				□ contrib is the name of the branch you are on in this example
		○ If you want to diff the last commit of the branch you are on and its common ancestor with another branch you can use the … syncatx.  For exmple
			§ $ git diff master…contrib
	
	Rebasing and Cherry-Picking Workflows
	
		○ A cherry pick in Git is like a rebase for a single commit.
			§ It takes the patch that was introduced in a commit and tries to reapply it on the branch you're currently on.  
			§ This is useful if you have a number of commits on a topic branch and you want to integrate only one of them, or if you have one commit on a topic branch and you'd prefer to cherry-pick it rather than run rebase.
			§ If you want to pull commit e43a6 into your master branch, you can run …
				□ $ git cherry-pick e43a6
	Rerere
	
		○ If you're doing lots of merging and rebasing, or you're maintining a long lived topic branch, Git has a feature called "rerere" that can help.
			§ Rerere stands for reuse recorded resolution.  it’s a way of shortcutting manual conflict resolution.  
			§ When rerere is enabled Git will keep a set of pre and post images from successful merges, and if it notices that there's a conflict that looks exactly like one you've already fixed, it'll just use the fix from last time, without bothering you with it.
	
	Generating a Build Number
	
		○ Git does not have monotonically increasing numbers like v123 to go with each commit.  If you want to have a human readable name to go with a commit, you can run git describe on that commit.  In response Git will generates as string consisting of the name of the most recent tag earlier than that commit, folllowed by the number of commits since that tag, followed by a partial SHA-1 value of the commit being described.  
			§ 
	
	Preparing a Release
	
		○ One thing you want to do is create an archive of the latest snapshot of yoru code for those who don't use Git.  The command to do this is …
			§ $ git archive master --prefix='project/' --format=zip > `git describe master`.zip
			§ This will give you a nice tarball and zip archive of your project relaesase that you can upload to your website or email to people.
	
	The Shortlog
	
		○ A nice way of quickly getting a sorot of changelog of what has been added to your project since your last release is to use the git shortlog command.  It  summarizes all the commits in the range you give it:
			§ [comment]: <> (TODO: This was a straight copy paste from notes so need to clean up formatting)
