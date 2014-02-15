### Introduction

So, you want to help us make Oozie better?  Our development model is, essentially, a pull based model with a consensus building phase modeled after the [Apache's voting process](http://www.apache.org/foundation/voting.html). We don't have a formal role of a committer, though (remember -- we are following a pull, not a push model for applying changes). In fact, according to our policy nobody can **ever** commit his or her own changes.

### What to consider before contributing

Most likely than not, your interest in contributing to Oozie comes from the need to scratch a proverbial itch. Of course, the best way to scratch your own itch is to let somebody else do that. Thus before [promoting](http://www.hostgatorpromocodez.com) anything else, consider:

1. subscribing yourself to the [oozie-users](http://groups.yahoo.com/group/Oozie-users/join) mailing list and make 
    sure you get all the feeds for Oozie from the GitHub
2. browsing through the list of open issues that we've got

### How to contribute

You're still convinced that Oozie needs to be modified in order to handle your problem. And you even have an idea on how to do that, well here's how to turn that idea into a commit (or two) in 8 easy steps:

1. Before you can have fun with code, we really need you (or the company you work for) to sign our [Yahoo! Open Source contributor license agreement](http://yahoo.github.com/oozie/Contribution%20License%20Agreement%20Yahoo.pdf). This is the only boring step -- we promise. ;-) 
2. Start with [creating an issue](http://oozie-jira.hadoop.developer.yahoo.net/browse/OOZIE) clearly describing:
   * the problem you're trying to solve
   * the outline of your proposed solution
3. Let the issue "*ferment*" for a little while and for more elaborate requests consider sending an email to the
    oozie-users mailing list (not all who are subscribed tend to follow us on GitHub). The point of all of this is
    to prevent you from wasting cycles on something that might not even be a problem at all.
4. [Fork Oozie](http://github.com/yahoo/oozie#fork_box) into your very own GitHub repository
5. Create a [topic branch](http://www.kernel.org/pub/software/scm/git/docs/gitworkflows.html#_topic_branches) with its name corresponding to the JIRA issue ID you were assigned when you opened the issue in step 2. E.g. if your issues happens to be #14 and your forked repo is at github.com/grace/oozie:

    $ git clone git@github.com/grace/oozie.git<br>
    $ cd oozie.git<br>
    $ git branch OOZIE-14 master<br>
6. Do all your work inside that topic branch and and make sure that all the commits inside that topic branch have the first line formatted like this: ```Closes OOZIE-<id> <issue description on JIRA>```. E.g if you happen to be working on the following issue [#14](http://github.com/yahoo/oozie/issues/closed#issue/14)

    $ git commit -m"Closes OOZIE-14 references SVN in bin/mkdistro.sh"<br><br>
should do the trick. This is important, because it lets GitHub do automatic crosslinking between commits and issues when your changes get accepted.

7. Once you're satisfied with the changes and you want the rest of Oozie developers to take a look at them and consider them for inclusion, push your changes back to your own repository and send us a Pull request. Remember to:
   * specify a branch on our end where your changes need to be applied to (most likely it'll be the default master, but if you're proposing changes to our web pages it'll be gh-pages and if you happen to be working on one of the sustaining branches it'll be one of those). 
   * start your pull request message with the following ```Closes OOZIE-14``` for the very same cross-linking purposes (continuing the example from above)
   * if the pull request is for any other branch but master you have to explicitly let us know that
   * finally, the pull request message is a good place to put answers to the questions you might be anticipating from the peer review

8. This is the moment where peer-review of your changes starts. It runs for at least 36 hours and the point of it is for at least 2 Oozie developers to give you a +1. Once you get 2 votes of +1 **and** no *clearly substantiated* -1 votes **and** the 36 hour window has expired your changes will get pulled into the Yahoo! repository by one of the developers who gave you +1 votes.  Most of the time you'll get questions, and sometimes even suggestions on [code](http://www.hostgatordiscountcodez.com) improvements, before the criteria above are met. At this point you'll have to answer the questions and/or change your code by committing additional deltas on top of your existing commits within the same branch. It is **extremely important**:
      * **NOT** to remove/replace the existing topic branch
      * **NOT** to remove/modify the existing commits on that branch