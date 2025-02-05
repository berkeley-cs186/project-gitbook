# Development Container Setup

A [development container](https://containers.dev/) is a [quasi-standard](https://github.com/devcontainers/spec) way to configure a development environment in a Docker container and install an agent that communicates with the IDE running on the host machine. The container can run locally or on a remote host.

This configuration solution is an option for students who are unable to configure their local environment via the existing Maven and JDK setup infrastructure and/or need to build on a remote host due to system constraints.

# Requirements

## Docker

Install [Docker](https://www.docker.com/) on the machine running the build container, i.e. your laptop or personal computer.

We recommend that you run the build container on the host machine (a personal device); however, it is possible to run the container elsewhere, though course staff is unlikely to support alternate methods.

## IDE

We recommend using IntelliJ IDEA due to its debugging capabilities and are unlikely to support other IDEs. If you prefer a different IDE, see below for instructions to run a development container via Visual Studio Code (VS Code) or GitHub Codespaces.

### IntelliJ IDEA Ultimate

_Note that this version is different from IntelliJ IDEA CE (Community Edition); however, both can remain installed and be used on your device simultaneously._

Development containers are currently supported only in the [Ultimate edition](https://www.jetbrains.com/idea/) of IntelliJ IDEA. You may obtain a free, non-commercial license [here](https://www.jetbrains.com/community/education/#students) on a new or existing JetBrains account using your university address (@berkeley.edu). Note that if you have an existing JetBrains account linked to a different email address, you may need to [link](https://sales.jetbrains.com/hc/en-gb/articles/7654171906706-Linking-multiple-email-addresses-to-your-JetBrains-Account) your personal email to your university email JetBrains account.

### Other IDEs

Install your preferred IDE. Note that course staff will only support IntelliJ IDEA Ultimate (recommended), VS Code, and GitHub Codespaces.

# Creating the Dev Container

## IntelliJ IDEA Ultimate

IntelliJ is currently configured to use the [EAP](https://www.jetbrains.com/idea/nextversion/) (early access) version of its IDE to connect to the container by default. The EAP version is the beta version of the IDE that is both less stable and on a license which expires after 30-45 days. To disable this, go to `Settings > Advanced Settings` and uncheck the box "Always use the latest backend" (see [here](https://youtrack.jetbrains.com/articles/SUPPORT-A-551) for the article).

![Screenshot - Uncheck "Always use the latest backend.".png](../images/dc-0.png)

Start the IDE and navigate to "Dev Containers":

![Screenshot - Select Dev Containers](../images/dc-1.png)

Create a "New Dev Container":

![Screenshot - Select New Dev Container](../images/dc-2.png)

Use the HTTPS address of the rookiedb repository for the relevant project. For example, a student with GitHub username `oski` working on Project 0 would use the repository address `https://github.com/cs186-student/sp25-proj0-oski.git`:

![Screenshot - Add Repository and Branch information](../images/dc-6.png)

Click "Build Container and Continue" and wait for Docker to download and build the image (this will take a few minutes).

### Troubleshooting

#### GitHub Authentication

GitHub [deprecated password-based authentication](https://github.blog/security/application-security/token-authentication-requirements-for-git-operations/) in July 2020, which means that your password no longer works for command-line authentication. If asked to authenticate your GitHub account via IntelliJ, create a [Personal Access Token (PAT)](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens#creating-a-personal-access-token-classic), which you may use to set up each of your dev containers. Ensure that you save your token securely, as it cannot be accessed again via your account.

#### SSH Repository URL

If you're creating a dev container using the SSH URL instead of the HTTPS URL, you may need to run `ssh-add -k` locally to add your private SSH key to your device. You may run `ssh-add -l` to see the SSH keys currently on your device.

## Visual Studio Code (VS Code) (local)

Select ![Remote Explorer](../images/dc-4.png) and click the ![plus icon](../images/dc-5.png) to create a new dev container.

Then select `Clone Repository in Container Volume` and follow prompts to clone the repository into the container.

## Codespaces (VS Code or web browser)

A [GitHub Codespace](https://github.com/features/codespaces) runs the build container in Azure, either in a browser or through the VS Code application.

Either open VS Code or the web-based editor for the repository, `https://github.dev/cs186-student/sp25-projX-username`, by navigating through `github.dev` instead of `github.com`.

Select ![Remote Explorer](../images/dc-4.png) and click on "Create Codespace."

Please consult the documentation for latest pricing information. GitHub currently provides 120 core-hours (60 hours in the least-capable image) per month to all users for free. If you elect to run in this or another remote configuration, course staff may be unable to provide support.

#

CS 186 Spring 2025

Authors: Chris Douglas, Janani Sriram