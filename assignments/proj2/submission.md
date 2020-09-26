# Submitting the Assignment

### Files

You may **not** modify the signature of any methods or classes that we provide to you, but you're free to add helper methods.

You should make sure that all code you modify belongs to files with `TODO(proj2)` comments in them \(e.g. don't add helper methods to DataBox\). A full list of files that you may modify follows:

* `src/main/java/edu/berkeley/cs186/database/index/BPlusTree.java`
* `src/main/java/edu/berkeley/cs186/database/index/InnerNode.java`
* `src/main/java/edu/berkeley/cs186/database/index/LeafNode.java`

Make sure that your code does _not_ use any \(non-final\) static variables -- this may cause odd behavior when running with the autograder vs. in your IDE \(tests run through the IDE often run with a new instance of Java for each test, so the static variables get reset, but multiple tests per Java instance may be run when using maven, where static variables _do not_ get reset\).

### Gradescope

Once all of your files are prepared in your repo you can submit to Gradescope through GitHub the same way you did for [Project 0](../proj0/submitting.md#pushing-changes-to-github-classroom). 

### Partners

If you haven't yet already be sure to fill out [this form](https://forms.gle/REQPugJuLDEK8hfr7) so we know who you're working with. When submitting to Gradescope remember to add your partner to the submission so they can get credit. Slip days will be deducted individually. For example: You submit on time, but your partner submits a day late. Your partner will have to use a slip day or will receive a late penalty on the project \(but you will not\).

### Grade breakdown

* 60% of your grade will be made up of tests released to you \(the tests that we provided in

  `database.index.*`\).

* 40% of your grade will be made up of hidden, unreleased tests that we will run on your submission after the deadline.

