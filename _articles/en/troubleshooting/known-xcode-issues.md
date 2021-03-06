---
changelog:
last_modified_at:
title: List of known Xcode issues
redirect_from:
- "/ios/known-xcode-issues/"
tag:
- xcode
- " troubleshooting"
- ios
description: 'Read about Xcode issues: performance problems, test failures, troubles
  with the Xcode Command Line Tools.'
menu:
  troubleshooting-main:
    weight: 16

---
**Feel free to add your comments to this list.**

[You can do it directly on GitHub, by clicking this link](https://github.com/bitrise-io/devcenter/blob/master/_articles/en/troubleshooting/known-xcode-issues.md), just don't forget to send it as a Pull Request ;)

## Performance related

_Note: mainly affects UI tests._

The root cause of the issue is that Xcode / the iOS Simulator has issues in performance limited environments. This included Virtual Machines (which is how your builds are running on [bitrise.io](https://www.bitrise.io)), MacBook Airs, Mac Minis with HDD storage, ...

It can happen even if you use [Apple's Xcode Bots CI server](http://www.openradar.me/23386199) on non SSD Mac Mini.

Examples:

* UI Tests fail to start
* One or more UI Test case hangs

Related links & reports:

* [XCode bot error: Early unexpected exit, operation never finished bootstrapping](http://stackoverflow.com/questions/36312500/xcode-bot-error-early-unexpected-exit-operation-never-finished-bootstrapping)
* [UI Testing Failure - Failed to launch within 2.5s, Interrupting test](https://forums.developer.apple.com/thread/20747)
* [Assertion Failure: UI Testing Failure - Failed to receive completion for ...](https://forums.developer.apple.com/thread/15209)
* [Assertion Failure: UI Testing Failure - Failed to receive completion for XCDeviceEvent: page 12 usage 64 duration 0.01s within 10.0s](https://forums.developer.apple.com/thread/31311)
* [UI Testing Failure - Failed to perform AX action for monitoring the event loop](https://forums.developer.apple.com/thread/31312)
* [UI Testing Failure due to failed AX Action](https://forums.developer.apple.com/thread/4472)
* [Every other test case being skipped - UI Testing Failure - App state for <XCUIApplicationProcess: ...> is XCApplicationStateRunningActive (3), still not XCApplicationStateNotRunning](https://forums.developer.apple.com/thread/28732)

### Possible solutions

* As reported [in this StackOverflow answer](http://stackoverflow.com/a/37866825/974381) & [in this Apple dev forum discussion](https://forums.developer.apple.com/thread/4472)
  a possible workaround can be to **not to store** `**XCUIApplication()**` **in a variable**, instead
  reference / use it directly. E.g. instead of: `let app = XCUIApplication() ; ... ; app.launch()`
  do: `XCUIApplication().launch()`
* Others reported that if you add a delay (`sleep(10)`) after every `app.launch()` related to your tests, it can
  leave enough time for Xcode / the iOS Simulator to initialize the Accessibility labels,
  so that UI Tests can properly find the elements by the Accessibility labels. ([Related Apple developer forum discussion](https://forums.developer.apple.com/thread/28732))
  * Related: remove every explicit `app.terminate()` in your `tearDown()` method(s)
* Try another Simulator device
* [Use the Async testing APIs](http://stackoverflow.com/a/32481202/974381)
* Some users had success with splitting the tests into multiple Schemes, and running those separately, with separate Test Steps.
  * A great article about splitting tests into multiple Schemes:
    [http://artsy.github.io/blog/2016/04/06/Testing-Schemes](http://artsy.github.io/blog/2016/04/06/Testing-Schemes)
* [Sometimes it's related to a code which makes Xcode to misbehave](https://github.com/fastlane/fastlane/issues/3874#issuecomment-219991408)
* Try another Xcode version.

## Xcode Unit Test fails without any error, with exit code 65

This can be caused by a lot of things, Xcode or some other tool simply
omits / does not present any error message.

Some possible solutions:

* First of all, if you use `xcpretty` to format the output try a build without it (if you use the Xcode Test Step you can set `xcodebuild` as the "Output Tool" option/input
  to not to format the log produced by `xcodebuild`). The cause is: `xcpretty` sometimes
  omits the error message in it's output.
* If you don't use our **Xcode Test** Step to run your UI Test you should try to run it with our Xcode Test step. We always try to improve the reliability of the step, implementing known workarounds for common issues.
* If you use our Xcode Test Step: make sure you use the latest version, as it might include additional workarounds / fixes.
* Try [another Xcode version](http://devcenter.bitrise.io/docs/available-stacks#section-how-to-switch-to-the-new-beta-stacks),
  there are issues which are present in one Xcode version but not in another one.
* Make sure your desired [Xcode scheme is shared](https://devcenter.bitrise.io/troubleshooting/frequent-ios-issues/#xcode-scheme-not-found). Don't forget to commit and push the changes if you just enabled it.
* It might also be a project configuration issue in your Xcode project, a code issue in your tests,or a multi-threading issue in your code.
* We received reports that this might also be caused by Code Coverage report generation,
  you can disable the **Generate code coverage files?** option of the Xcode Test Step to not to generate Code Coverage files.

## Segmentation fault

The error is:

    clang: error: unable to execute command: Segmentation fault: 11

This is usually due to Xcode version mismatch - that you use a different Xcode on your Mac than the one you use on [bitrise.io](https://www.bitrise.io).

The solution is simple, just make sure that you use the same Xcode version everywhere.

## Issues which occur with the Xcode Command Line Tools (`xcodebuild`), but not with Xcode.app

### xcodebuild hangs when a test causes EXC_BAD_ACCESS kernel exception

[xcodebuild hangs when a test causes EXC_BAD_ACCESS kernel exception](https://openradar.appspot.com/24222858)

Note: this can happen only on specific iOS Simulators / iOS versions too, e.g. if the app only crashes on iOS 8, but not on iOS 9

### Xcode 8 - xcodebuild .. test hangs at the very end of the tests

`xcodebuild .. test` hangs at the end of the tests (after it printed the summary of the tests)
if the output of `xcodebuild ..` is piped / redirected in any way.
This means that `xcodebuild .. test .. | xcpretty` or even `tee` can be used to reproduce this issue.

* Related [radar](http://openradar.appspot.com/26872644) and [xcpretty](https://github.com/supermarin/xcpretty/issues/227) issues.
* Workaround: use a **Script** Step instead of the Xcode Test step, and copy paste the `xcodebuild` command from the hanging Xcode Test Step's log, without `| xcpretty` etc. Of course, with this you won't be able to use the built in features the Xcode Test Step provides, but the base `xcodebuild` command should be able to run, if the output
  is not redirected / piped.

#### Every/Any Xcode command hangs

This is a rare issue, caused by running a **non shared Scheme**.

`xcodebuild` can only work with **shared Schemes** and user schemes (auto created by Xcode.app).
`xcodebuild`, unlike Xcode.app, **does not** auto create user schemes, it can only work with
**shared schemes** and already existing user schemes (Xcode.app creates the user scheme when you open
the project in Xcode.app **on the specific Mac machine the first time**, for schemes which are not marked as shared).
If you try to run a command on a missing / non shared Scheme it usually manifests in a "scheme not found"
error, but we saw projects where it resulted in `xcodebuild` hanging, instead
of an error message.

If this is the case then any `xcodebuild` command will hang, even something
as simple as `xcodebuild -list`.

##### Solution

[Make sure that you marked the Scheme as shared, and that you actually committed & pushed it into your repository](https://devcenter.bitrise.io/troubleshooting/frequent-ios-issues/#xcode-scheme-not-found).

## Build hangs

### Simulator reset

As reported [here](https://github.com/bitrise-io/steps-xcode-test/issues/57#event-796203051), if you do a simulator reset during the build, e.g. with a pre-action Build Phase Script `xcrun simctl erase all`, it can cause Xcode / the Simulator to hang.

### Other

It might also not be Xcode related, but might be caused by something in your project when it runs in an Xcode Step (Xcode Test, Xcode Archive, ...). For example if you have a Run Phase Script in your Xcode project, that will run during Xcode build/test/archive, and that script hangs for some reason (e.g. it waits for a user input).

Check out our guide on [frequent iOS issues](https://devcenter.bitrise.io/troubleshooting/frequent-ios-issues/) for more information!

{% include banner.html banner_text="Now you know everything" url="https://app.bitrise.io/dashboard/builds" button_text="Go to Bitrise now" %}