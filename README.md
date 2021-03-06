Trac - GitHub integration
=========================

Features
--------

This Trac plugin performs two functions:

- update the local git mirror used by Trac after each push to GitHub, and
  notify the new changesets to Trac;
- replace Trac's built-in browser by GitHub's (optional).

The notification of new changesets is strictly equivalent to the command
described in Trac's setup guide:

    trac-admin TRAC_ENV changeset added ...

This feature set is comparable to https://github.com/davglass/github-trac.

However trac-github has the following advantages:

- it supports multiple repositories;
- it uses GitHub's generic WebHook URLs;
- it has no external dependencies;
- it is well documented — more docs than code;
- it has an extensive test suite — three times more tests than code;
- it has a much smaller codebase;
- it makes better use of Trac's APIs.

Requirements
------------

trac-github requires Trac >= 0.12 and the git plugin.

This plugin [is included](http://trac.edgewall.org/wiki/TracGit) in Trac >=
0.13 — you only have to enable it in `trac.ini`. For Trac 0.12 you have to
[install it](http://trac-hacks.org/wiki/GitPlugin):

    pip install -e git://github.com/hvr/trac-git-plugin.git#egg=TracGit-dev

Then install trac-github itself:

    pip install -e git://github.com/aaugustin/trac-github.git#egg=TracGitHub-dev

Setup
-----

_Warning: the commands below are provided for illustrative purposes. You'll
have to adapt them to your setup._

First, you need a mirror of your GitHub repository, writable by the webserver,
for Trac's use:

    cd /home/trac
    git clone --mirror git://github.com/<user>/<project>.git
    chown -R www-data:www-data <project>.git

Ensure that the user under which your web server runs can update the mirror:

    su www-data
    git --git-dir=/home/trac/<project>.git remote update --prune

Now edit your `trac.ini` as follows to configure both the git and the
trac-github plugins:

    [components]
    trac.versioncontrol.web_ui.browser.BrowserModule = disabled
    trac.versioncontrol.web_ui.changeset.ChangesetModule = disabled
    trac.versioncontrol.web_ui.log.LogModule = disabled
    tracext.git.* = enabled
    tracext.github.* = enabled

    [github]
    repository = <user>/<project>

    [trac]
    repository_dir = /home/trac/<project>.git
    repository_type = git

Reload the web server and your project should appear in Trac.

Now go to your project's administration page on GitHub. In the service hooks
tab, select WebHook URLs. Enter the URL of your Trac installation followed by
`github`, like this:

    http://<trac.example.com>/github

You may want to restrict access this URL to GitHub's IPs. They're listed just
under the form WebHook URLs setup form.

Branches
--------

By default, trac-github notifies all the commits in each changeset to Trac.
When you're merging a branch, the commits on the branch are notified again.
This can result in duplicate ticket updates and notification emails. To avoid
this, you can configure trac-github to only notify commits on some branches:

    [github]
    branches = master

You can provide more than one branch name, and you can use [shell-style
wildcards](http://docs.python.org/library/fnmatch):

    [github]
    branches = master stable/*

Multiple repositories
---------------------

If you have multiple repositories, you must tell Trac how they're called on
GitHub:

    [github]
    repository = <user>/<project>               # default repository
    <reponame>.repository = <user>/<project>    # for each extra repository
    <reponame>.branches = <branches>            # optional

When you configure the WebHook URLs, append the name used by Trac to identify
the repository:

    http://<trac.example.com>/github/<reponame>

Advanced use
------------

trac-github provides two components that you can enable separately.

- **`tracext.github.GitHubPostCommitHook`** is the post-commit hook called by
  GitHub. It updates the git mirror used by Trac, triggers a cache update and
  notifies components of the new changesets. Notifications are used by Trac's
  [commit ticket updater](http://trac.edgewall.org/wiki/CommitTicketUpdater)
  and [notifications](http://trac.edgewall.org/wiki/TracNotification).
- **`tracext.github.GitHubBrowser`** replaces Trac's built-in browser by
  redirects to the corresponding pages on Github. Since it replaces standard
  URLs of Trac, if you enable this pluign, you must disable three components in
  `trac.versioncontrol.web_ui`, as shown in the configuration file above.

License
-------

This plugin is released under the BSD license.

It was initially written for [Django's Trac](https://code.djangoproject.com/).
