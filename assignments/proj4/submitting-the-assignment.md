# Submitting the Assignment

## Files

You may **not** modify the signature of any methods or classes that we provide to you, but you're free to add helper methods.

You should make sure that all code you modify belongs to files with `TODO(proj4_part1)` and `TODO(proj4_part2)` comments in them \(e.g. don't add helper methods to DataBox\). A full list of files that you may modify follows:

* `src/main/java/edu/berkeley/cs186/database/concurrency/LockType.java`
* `src/main/java/edu/berkeley/cs186/database/concurrency/LockManager.java`
* `src/main/java/edu/berkeley/cs186/database/concurrency/LockContext.java` \(Part 2 only\)
* `src/main/java/edu/berkeley/cs186/database/concurrency/LockUtil.java` \(Part 2 only\)
* `src/main/java/edu/berkeley/cs186/database/table/Table.java` \(Part 2 only\)
* `src/main/java/edu/berkeley/cs186/database/table/PageDirectory.java` \(Part 2 only\)
* `src/main/java/edu/berkeley/cs186/database/memory/Page.java` \(Part 2 only\)
* `src/main/java/edu/berkeley/cs186/database/Database.java` \(Part 2 only\)

Make sure that your code does _not_ use any static \(non-final\) variables - this may cause odd behavior when running with maven vs. in your IDE \(tests run through the IDE often run with a new instance of Java for each test, so the static variables get reset, but multiple tests per Java instance may be run when using maven, where static variables _do not_ get reset\).

## Gradescope

Once all of your files are prepared in your repo you can submit to Gradescope through GitHub the same way you did for [Project 0](../proj0/submitting.md#pushing-changes-to-github-classroom).

### Submitting via upload <a id="submitting-via-upload"></a>

If your GitHub account has access to many repos, the Gradescope UI might time out while trying to load which repos you have available. If this is the case for you, you can submit your code directly using via upload. You can zip up your source code with `python3 zip.py --assignment proj3` and submit that directly to the autograder.

## Partners

Only one partner has to submit, but please make sure to add the other partner to the Gradescope submission. Slip minutes will be calculated individually. For example, if partner A has 10 slip minutes remaining and partner B has 20 slip minutes remaining and they submit 20 minutes late, partner A will be subject to the late penalty (1/3 off partner A's score) while partner B will have 0 remaining slip minutes and no late penalty applied to partner B's score. If you wish to make individual submissions at separate times, please fill out [this form](https://docs.google.com/forms/d/e/1FAIpQLScUcuBNesd984X1I_3RbXGDxmS5v2pebTS_sF2QrbuvAK1sgg/viewform?usp=sf_link) so you aren't flagged for academic dishonesty.

## Grading

* Your submission for Part 1 will be worth 20% of your Project 4 grade and will come entirely from the Part 1 public tests.
* Your submission for Part 2 will be worth 80% of your Project 4 grade.
  * 5% for Part 1 hidden tests
  * 60% for Part 2 public tests
  * 15% for Part 2 hidden tests

Your Part 2 submission will be used to run all hidden tests \(and public Part 2 tests\). If you do not have a submission for Part 2, you will receive a 0% on all hidden tests for Part 1 and 2. You may go back and make changes to Part 1 even after that part deadline has passed before submitting to Part 2.

