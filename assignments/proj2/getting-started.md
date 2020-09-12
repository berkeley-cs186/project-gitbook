# Getting Started

## Logistics

This project is due **Thursday, 9/24/2020 at 11:59PM PDT \(GMT-7\)**. It is worth 6% of your overall grade in the class. The workload for the project is designed to be completed solo, but this semester we're allowing students to work on this project with a partner if they want to. Feel free to search for a partner on [this Piazza thread](https://piazza.com/class/kducz9b1i3h78i?cid=5)!

## Prerequisites

You should watch the B+ Trees lecture before beginning this project. Later parts will require material from the Indices and B+ Trees Refinements lecture.

## Academic Integrity Policy

“_As a member of the UC Berkeley community, I act with honesty, integrity, and respect for others._”  
— UC Berkeley Honor Code

**Read through the academic integrity guidelines** [**here**](https://piazza.com/class/kducz9b1i3h78i?cid=42)**.** We will be running plagiarism detection software on every submission against our own database of this semester's submissions, past submissions, and publicly hosted implementations on platforms such as GitHub and GitLab, followed by a thorough manual review process. Plagiarism on any assignment will result in a [non-reportable warning](https://sa.berkeley.edu/student-code-of-conduct-section6) and a grade penalty based on the severity of the infraction.

As long as you follow the guidelines there isn't anything to worry about here. While we do rely on software to find possible cases of academic dishonesty every case is reviewed by multiple TAs who can filter out false positives.

## Fetching the released code

The GitHub Classroom link for this project is in the Project 2 release post on [Piazza](https://piazza.com/class/kducz9b1i3h78i). Once your private repo is set up clone the project 2 skeleton code onto your local machine. You'll be working off of a fresh copy of the MOOCBase skeleton instead of reusing the one from project 0.

### Setting up your local development environment

If you're using IntelliJ you can follow the instructions [in project 0](../proj0/getting-started.md#setting-up-your-local-development-environment) in to set up your local environment again. Once you have your environment set up you can head to the next section [Your Tasks](your-tasks.md) and begin working on the assignment.

## Adding a partner

If you choose to work with a partner and want to share your code over GitHub you'll need to pick between the two of you who's copy of the starter code you want to work off of. For example, if users `OskiBear` and `JoeBruin` want to collaborate then they might both choose to work off of `fa20-proj2-OskiBear`. In that case `OskiBear` would navigate to "Settings", "Manage access", then "Invite teams or people" and add `JoeBruin`\(see images below\). `JoeBruin` should receive an email afterwards containing an invitation to the repo. Alternatively `OskiBear` can share the link to the repo after sending the invitation.

Once you both have access to the repo you can share files over git, pushing and pulling updates as you collaborate. **Do not add anyone other than your project partner to your repository.** You aren't allowed to share code with any students other than your project partner for this project.

![Navigate to your fa20-proj2-yourname repo&apos;s Settings](../../.gitbook/assets/image%20%287%29.png)

![Go to Manage Access](../../.gitbook/assets/image%20%286%29.png)

![](../../.gitbook/assets/image%20%288%29.png)

![Invite your partner \(replace JoeBruin with your partner&apos;s GitHub username\)](../../.gitbook/assets/image%20%289%29.png)

## Debugging Issues with GitHub Classroom

Feel free to skip this section if you don't have any issues with GitHub Classroom. If you are having issues \(i.e. the page froze or some error message appeared\), first check if you have access to your repo at `https://github.com/berkeley-cs186-student/fa20-proj2-username`, replacing `username` with your GitHub username. If you have access to your repo and the starter code is there, then you can proceed as usual. If you have access to your repo but the starter code is not there, run the following commands in a terminal \(again replacing `username` with your GitHub username\):

```text
git clone https://github.com/berkeley-cs186/fa20-moocbase fa20-proj2
cd fa20-proj2/
git remote remove origin
git remote add origin https://github.com/berkeley-cs186-student/fa20-proj2-username.git
git push -u origin master
```

Then, you can proceed as usual.

### 404 Not Found

If you're getting a 404 not found page when trying to access your repo, make sure you've set up your repo using the GitHub Classroom link in the Project 2 release post on [Piazza](https://piazza.com/class/kducz9b1i3h78i).

If you don't have access to your repo at all after following these steps, feel free to contact the course staff on Piazza.

