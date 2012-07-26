---
layout: post
title: "Amend Message/Author at a Specific Commit in Git"
description: ""
category: configuration
tags: [git]
---
{% include JB/setup %}


# The Skinny #

As a Mercurial user trying to learn the ropes of Git, especially since this site uses Git for version control, there are bound to be mistakes here and there. Thankfully, Git provides the means to go back in history and re-write it using the **rebase** command.

Note that this is used for editing commit messages or authors only.


# Sample Scenario #

Say you have a commit history of A-B-C-D-E. You realised that you forgot to include some details for the commit message for B, and you want to change the author name for C. So:

1. Go back to parent of the commit that you wish to edit. In this case, we want to edit B, so go back to A.
		
		$ git rebase -i <hash of A>

2. The default text editor will come up, containing the list of subsequent commits (B to E). For commits that you wish to edit, change the option from **pick** to **edit**.

3. Save and quit the text editor.

4. We want to edit the commit message for B, so type the following:

 		$ git commit --amend

5. The text editor will come up again, and we can now proceed to add the missing details/spelling mistakes etc. Save and quit.

6. We're now at C, of which we want to change the name of the author. So type the following:

		$ git commit --amend --author="Author Name <email@address.com>"

7. The text editor will come up again. Since we're not editing the commit message, simply save and quit.

8. We are now done with the editing, so type the following:

		$ git rebase --continue

9. The rebase is now complete.

10. The new history will appear as a new branch. For clarity sake, I usually just create a new clone of the repository, which will remove the old history branch. There is probably a git command for it. Not sure as of now.


# Reference #

[Change commit author at one specific commit - Stack Overflow](http://stackoverflow.com/a/3042512)
