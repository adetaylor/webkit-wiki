The WebKit project has a number of expectations for how pull requests are formatted. These expectations are codified in `Tools/Scripts/git-webkit pr`, which contributors can run after drafting a change locally to configure a pull request. The WebKit project recommends you follow [this guidence](/WebKit/WebKit/wiki/Contributing#contributing-code) and leverage our tools. If contributors would like to use alternative tools, this document explains what `Tools/Scripts/git-webkit pr` is doing and how WebKit expects pull requests to be formatted.

## Bug Tracking

The first step of most pull requests is creating a [bug](https://bugs.webkit.org/enter_bug.cgi). While it is not expected that every pull request has a unique bug, it is expected that every landed commit can be linked back to a bug report. In particular, pull requests which are regressions or follow-up fixes often reference the bug the original commit references.

## Branching

Pull request branches are owned by their author, which is why `git-webkit setup` [creates a personal fork of WebKit](/WebKit/WebKit/wiki/Contributing#setup). This means that the WebKit project cannot enforce branching idioms, although there are some suggestions the WebKit team has so that other contributors can more easily access proposed changes. `Tools/Scripts/git-webkit pr` derives it's `eng` prefixed branch from the bug title of the bug associated with a pull request.

We suggest that pull request branch names are prefixed by `eng/` or `dev/` so that contributors are clear which branches contain production code when they add other user's forks as remotes. Notably, [EWS](https://ews-build.webkit.org) is unable to apply changes which come from the branch they are targeting (ie, [EWS](https://ews-build.webkit.org) cannot apply a change from `Contributor/WebKit:main` onto `WebKit/WebKit:main`), so in order to be reviewed, changes must come from a different branch.

## Commit Messages

The WebKit project heavily relies on commit messages to defend project performance and correctness along with using them to manage releases. In support of this, the WebKit project mandates the following in every commit message:

* Bug title
* Bug url
* Reviewer (or explicit reason why a change is unreviewed)
* High level explanation (optional)
* Files changes, what was changed, and why

`git-webkit setup` [configures `.git/prepare-commit-msg`](/WebKit/WebKit/wiki/Contributing#setup) such that your commit message template is formatted to the standards of the WebKit project.

In addition to pull request commits having verbose commit messages, the WebKit project also expects the content of commit messages in the pull request description. This is so that reviewers can provide feedback on the commit message itself.

## Reviewing

Commits generally require the approval of a [reviewer](https://webkit.org/team/#reviewers), although there are narrow exceptions for test expectation gardening and build fixes. Reviewers will approve pull requests through the GitHub UI by marking pull requests as "Approved." Note that approval from someone who is not a reviewer will not be recognized by [Merge-Queue](/WebKit/WebKit/wiki/Contributing#merge-queue).

## Landing

Only repository administers have direct commit access, and this is reserved for repairing infrastructure issues. Pull requests should be landed through [Merge-Queue](/WebKit/WebKit/wiki/Contributing#merge-queue), which is achieved by applying [`merge-queue`](https://github.com/WebKit/WebKit/labels?q=merge-queue) or [`unsafe-merge-queue`](https://github.com/WebKit/WebKit/labels?q=unfsafe-merge-queue) label to your pull request.

[Merge-Queue](/WebKit/WebKit/wiki/Contributing#merge-queue) will check to make sure a change is reviewed by adding the name of all [reviewers](https://webkit.org/team/#reviewers) who have approved a pull request to the commit message. [Merge-Queue](/WebKit/WebKit/wiki/Contributing#merge-queue) will then check the commit message for `Reviewed by`.

Note that [Merge-Queue](/WebKit/WebKit/wiki/Contributing#merge-queue) will reject pull requests that are labeled by contributors who are not [committers](https://webkit.org/team/#committers).

## Test Gardening
Since direct commit access is limited only to repository administers, this will change the prior workflow of Test Gardening/Setting test expectations. As such the new process is outlined as follows: 

1. From a clean up-to-date checkout, create a new branch using`Tools/Scripts/git-webkit branch`. It will ask you for either your bug URL, or for just a title. Giving it the bug URL will title the branch the same as the title of your bug. You can manually title it as well, if you prefer. 

2. Mark your expectations/ make your changes, followed by 'git add .' and 'git commit'  Changelogs are no longer needed for this, but you will still need to add a commit message.

3. Your commit message **MUST** contain the term '**Unreviewed**' in order for your pull-request to be committed successfully. It is generally recommended that you use the term 'Unreviewed test gardening.' in place of 'reviewed by nobody. (OOPS)' for any commit that is setting any kind of test expectations.

An example template for your commit message looks like:
 
<img width="382" alt="Screen Shot 2022-05-19 at 4 49 23 PM" src="https://user-images.githubusercontent.com/57686024/169422157-1d9444ec-a69f-49ef-9172-81a961161887.png">

4.Once your expectations and commit message have been added, you can then submit your pull-request by running `Tools/Scripts/git-webkit pr`. from there your pull-request will be created, and submitted to [WebKit Pull Requests](https://github.com/WebKit/WebKit/pulls). A link directly to your pull-request will be in your terminal output.

5. Go to your pull-request with the provided link. Then on the right side of the page under the `labels` tab you will add `unsafe-merge-queue` just like the example below:
<img width="307" alt="Screen Shot 2022-05-19 at 4 38 14 PM" src="https://user-images.githubusercontent.com/57686024/169421389-38e57a73-2ba1-4a0e-baa5-4b610e41d117.png">

6. After adding the `unsafe-merge-queue` label, your pull-request will attempt to be committed. This should take 1-3 minutes. If there is an issue with the pull-request then the commit may fail with an error. 