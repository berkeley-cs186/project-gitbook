# Submitting the Assignment

## Files

You may **not** modify the signature of any methods or classes that we provide to you, but you're free to add helper methods.

You should make sure that all code you modify belongs to files with `TODO(proj5)` comments in them \(e.g. don't add helper methods to DataBox\). A full list of files that you may modify follows:

* `src/main/java/edu/berkeley/cs186/database/recovery/ARIESRecoveryManager.java`

Make sure that your code does _not_ use any static \(non-final\) variables - this may cause odd behavior when running with maven vs. in your IDE \(tests run through the IDE often run with a new instance of Java for each test, so the static variables get reset, but multiple tests per Java instance may be run when using maven, where static variables _do not_ get reset\).

## Gradescope

Once all of your files are prepared in your repo you can submit to Gradescope through GitHub the same way you did for [Project 0](../proj0/submitting.md#pushing-changes-to-github-classroom).

## Partners

If you haven't yet already make sure [this form](https://forms.gle/HFBF9evuhqriL6Cf9) has the correct partner information so we know who you're working with. **We're no longer allowing partner submissions on Gradescope**. If you worked with a partner you'll both need to submit on your own \(you're still free to submit identical code if you like though\). Slip days will be deducted individually. For example: You submit on time, but your partner submits a day late. Your partner will have to use a slip day or will receive a late penalty on the project \(but you will not\).

## Submitting via upload <a id="submitting-via-upload"></a>

If your GitHub account has access to many repos, the Gradescope UI might time out while trying to load which repos you have available. If this is the case for you, you can submit your code directly using via upload. You can zip up your source code with `zip -r submission.zip src/` and submit that directly to the autograder.

## Grading

* $public-weight$ of your grade will be made up of tests released to you \(the tests that we provided in `database.recovery.TestRecoveryManager`\).

* $hidden-weight$ of your grade will be made up of hidden, unreleased tests that we will run on your submission after the deadline.