The contributing guidelines outlined here are for a future GitHub based workflow. Up-to-date WebKit contribution guidelines are best outlined in our [ReadMe](https://github.com/WebKit/WebKit#readme).

## Checking Out WebKit

WebKit can be checked out via GitHub's https remote with:
```
git clone https://github.com/WebKit/WebKit.git WebKit
```

Or, if a [ssh key](https://github.com/settings/keys) has already been added to your GitHub profile:
```
git clone git@github.com:WebKit/WebKit.git WebKit
```

For more information about alternate WebKit remotes, consult [Alternate Remotes](/WebKit/WebKit/wiki/Git-Config#Alternate-Remotes)

## Setup

### `git-webkit`

WebKit provides a number of scripts in [Tools/Scripts](https://github.com/WebKit/WebKit/tree/main/Tools/Scripts) to aid in development. We recommend putting [Tools/Scripts](https://github.com/WebKit/WebKit/tree/main/Tools/Scripts) on your `PATH`. In particular, if [Tools/Scripts](https://github.com/WebKit/WebKit/tree/main/Tools/Scripts) is integrated into your `PATH`, the [git-webkit](https://github.com/WebKit/WebKit/tree/main/Tools/Scripts/git-webkit) script, which provides various programs for interaction with the WebKit repository, can be invoked as `git webkit`.

### `git-webkit setup`

The `setup` sub-command of [git-webkit](https://github.com/WebKit/WebKit/tree/main/Tools/Scripts/git-webkit) configures your local WebKit checkout for contributing code to the WebKit project. This script will occasionally prompt the user for input. The script does the following:

* Set your [name](/WebKit/WebKit/wiki/Git-Config#username) and [email address](/WebKit/WebKit/wiki/Git-Config#useremail) for the WebKit repository
* [Make Objective-C diffs easier to digest](/WebKit/WebKit/wiki/Git-Config#diff)
* Setup a commit message generator
* Set an [editor for commit messages](/WebKit/WebKit/wiki/Git-Config#coreeditor)
* Store a [GitHub API token](https://github.com/settings/tokens) in your system credential store
* Configure `git` to use the [GitHub API token](https://github.com/settings/tokens) when prompted for credentials, if using the HTTPS remote
* Create a [user owned fork](/WebKit/WebKit/wiki/Git-Config#Forking) of the WebKit repository

## Contributing Code

https://webkit.org/contributing-code/ outlines how to build and test WebKit along with code style guidelines and testing policies. Also see our [Safer C++ Guidelines](https://github.com/WebKit/WebKit/wiki/Safer-CPP-Guidelines).

Once a bug has been prepared and a code change drafted locally, contributors should run `git-webkit pr` to automatically generate a pull request. That script will do a few things:

* Create a `eng/` prefixed pull-request branch, if needed (e.g. `eng/constants-buffer` branch)
* Create a commit with locally modified files, if needed
* Rebase the pull-request branch against the latest version of its parent branch
* Push the pull-request branch to a user's personal fork of the project
* Create (or update) a pull-request to merge to the parent branch in WebKit

Note that the same process is used to update an already published pull-request. For a detailed breakdown on the expected format of WebKit pull requests, see [Pull Requests](/WebKit/WebKit/wiki/Pull-Requests).

### Code Review: Updating a PR

Make sure you're on the right branch. Make the necessary changes in your source tree. When you're ready, run `git-webkit pr` again to update the PR.

Before being landed by a [committer](https://github.com/orgs/WebKit/teams/committers), code must be reviewed by a [reviewer](https://github.com/orgs/WebKit/teams/reviewers). After a change is approved (sometimes through an `r+` or `r=me` in pull-request comments), it's the responsibility of the commit author to be sure that the change will not fail any EWS queues, this is not automatically enforced for most queues to speed up development.

## Landing Changes

### Merge-Queue

To land a pull request, add the [`merge-queue`](https://github.com/WebKit/WebKit/labels?q=merge-queue) or [`unsafe-merge-queue`](https://github.com/WebKit/WebKit/labels?q=unfsafe-merge-queue) label to your pull request. These labels will put your pull request into the [Merge-Queue](https://ews-build.webkit.org/#/builders/74) and [Unsafe-Merge-Queue](https://ews-build.webkit.org/#/builders/75), respectively, which will commit your pull request to the WebKit repository

[Unsafe-Merge-Queue](https://ews-build.webkit.org/#/builders/75) inserts reviewer information into a commit's message and modified change logs. We then check to ensure that a pull request has been reviewed by checking the commit message before landing the change. [Unsafe-Merge-Queue](https://ews-build.webkit.org/#/builders/75) _does not_ validate that a pull request builds.

Along with the actions performed by [Unsafe-Merge-Queue](https://ews-build.webkit.org/#/builders/75), [Merge-Queue](https://ews-build.webkit.org/#/builders/74) will validate that a pull request builds and run layout tests before landing the change.

### `git-webkit land`

_Landing should be achieved via merge-queue, this outlines the current behavior of `git-webkit land`_

To land a change, run `git-webkit land` from the branch to be landed. Note that only a [committer](https://github.com/orgs/WebKit/teams/committers) has the privileges to commit a change to the WebKit repository. `git-webkit land` does the following:

* Check to ensure a pull-request is approved and not blocked
* Insert reviewer names into the commit message
* Rebase the pull-request against its parent branch
* [Canonicalize](https://github.com/WebKit/WebKit/wiki/Source-Control#canonicalization) the commits to be landed
* Update the pull-request with the landed commit

## Running Nightly Builds

Download the appropriate build for your platform, updating the revision number. You may need to go back a few commits to find a build:
```
AS: https://s3-us-west-2.amazonaws.com/archives.webkit.org/mac-ventura-x86_64%20arm64-release/258xxx@main.zip
Intel: https://s3-us-west-2.amazonaws.com/archives.webkit.org/mac-ventura-x86_64%20arm64-release/258xxx@main.zip
```
Then:
```
cd 258xxx@main
sudo xattr -r -d com.apple.quarantine .
```

From here, you can try running Safari:
```
run-webkit-archive
```
A more stable way to run the nightly is to run MiniBrowser:
```
export WK=/Users/user/Downloads/258xxx@main/Release/ && DYLD_FRAMEWORK_PATH=$WK DYLD_LIBRARY_PATH=$WK __XPC_DYLD_FRAMEWORK_PATH=$WK __XPC_DYLD_LIBRARY_PATH=$WK $WK/MiniBrowser.app/Contents/MacOS/MiniBrowser
```