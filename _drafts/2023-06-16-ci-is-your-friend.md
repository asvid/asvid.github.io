---
layout: post
title: "Continuous Integration and Deployment: A Developer's Best Friends"
date: "2023-06-16 17:34"
description: "
In the fast-paced world of software development, Continuous Integration and Deployment (CI/CD) have emerged as game-changing tools for developers. Discover how CI/CD can be your ultimate allies, streamlining workflows, automating tests, and boosting collaboration. From user-friendly services like CircleCI and Bitrise to the robust capabilities of GitLab and GitHub Actions, we'll guide you through the best options to supercharge your development process.
"
permalink: "2023-06-16-ci-is-your-friend"
comments: true
toc: true
tags:
  - CI/CD
  - Continuous Integration
  - Continuous Deployment
  - Software Development
  - Automation
  - DevOps
  - Software Delivery
  - Testing
  - Code Quality

categories:
  - programming

image: /assets/posts/adapter.jpg

---

## Intro

In today's fast-paced software development landscape, Continuous Integration and Continuous Deployment (CI/CD) have become indispensable tools for developers. If you are already familiar with this concept, this post may not be for you. However, if you're new to CI/CD and want to learn what it is, how to set it up, and why it's cool, you're in the right place. CI/CD can eliminate some manual labor, streamline workflows, and make your development process more efficient and reliable.

## Understanding CI/CD

Continuous Integration (CI) and Continuous Deployment (CD) are development practices that aim to automate and streamline the process of building, testing, and deploying software. CI involves regularly merging code changes from multiple developers into a shared repository, where automated tests are executed to ensure smooth integration of changes. CD extends CI by automating the deployment process, enabling developers to release their applications quickly and reliably. CI provides a common build environment for your code, allowing team members to work on Linux, macOS, or Windows based on their preferences.

You can think of CI/CD services as machines that are there for you 24/7 to build, test, release, or perform any scheduled task without you having to explicitly instruct them. You don't have to maintain these machines in terms of hardware or internet access; it's all sorted for you. Your job is to script their work, set up triggers, and establish connections to external integrations.

### Free CI/CD Services

For junior developers starting their CI/CD journey, several free services offer user-friendly and efficient solutions. Here are three popular options worth exploring:

- [CircleCI](https://circleci.com): CircleCI provides a cloud-based CI/CD platform that seamlessly integrates with popular version control systems. With its intuitive interface and robust feature set, CircleCI allows you to easily set up automated workflows, run tests, and deploy applications.

- [Bitrise](https://bitrise.io): Bitrise is a CI/CD platform specifically designed for mobile app development. It offers a range of pre-configured workflows and integrations with popular development tools, making it an excellent choice for junior developers working on mobile projects.

- [GitLab](https://docs.gitlab.com/ee/ci/): GitLab provides built-in CI/CD capabilities within its platform. It offers a complete DevOps solution, including source code management, issue tracking, and CI/CD pipelines.

But lately, my favorite one is:

- [GitHub Actions](https://docs.github.com/en/actions): Built into the GitHub platform, GitHub Actions offers a flexible and powerful CI/CD solution. With Actions, you can define workflows using YAML, execute tests, and automate releases, all within your repository. Its seamless integration with GitHub makes it an ideal choice for developers already using the platform.

CircleCI and Bitrise allow you to set up your CI using user-friendly wizards. On the other hand, GitHub requires you to use `yaml` config files, which may seem intimidating at first, but it is also faster and kept inside your repository. A disadvantage of GitHub is the lack of a nice way to store secret files like keys. You can only store text data as secrets, so you have to encode your keystore to `base64` and decode it during the pipeline run.

While Jenkins is another free option, it is not cloud-based, so it may not be the best choice for beginners.

## Leveraging CI in Projects

CI offers several advantages for developers, whether working solo or in a team environment. Let's explore some key benefits:

### Automated Testing

CI enables developers to run automated tests, ensuring that any new changes or additions won't introduce regressions. By catching bugs early, developers can address issues promptly and maintain a stable codebase.

CI won't write your tests for you, but it ensures that tests are executed for every developer on your team, every time they want to merge their code. I remember working on a system so complex to set up locally that I was not even able to run tests on my machine. I was just writing tests and pushing changes so CI could do its job :)

### Enhanced Collaboration

CI tools can integrate with messaging platforms like Slack, enabling automated notifications to team members regarding build status, test results, and deployments. This fosters effective communication, keeps everyone in the loop, and promotes a collaborative development environment.

If you ever wrote a message to your team channel, so they know there is a PR for review, CI could do this for you. With proper setup, if the build after merge fails, the author of the faulty PR can also get a message to fix bugs.

### Streamlined Code Reviews

CI can automatically trigger tests, static code analysis, and lint checks when pull requests are created or updated. This handles the mundane parts of code reviews, checking for typos, code conventions, etc., allowing for quick feedback on code quality and ensuring that potential issues are identified early. Human reviewers can focus on more creative aspects of the review. This automation saves developers from manual reviews and reduces the chance of introducing code style or formatting issues. While you should run these tools locally before committing your code, CI acts as a bodyguard, providing an additional layer of assurance.

### Dependency Management

CI can automate the process of updating dependencies in your projects. By monitoring package releases and automatically bumping versions, developers can keep their dependencies up to date, ensuring security patches and performance improvements are readily available.

### Secure Storage for Signing Builds

Developers shouldn't have to keep signing keys for their software. By using CI to store and use signing keys, all team members can create signed builds (e.g., Android app) without the risk of losing the key due to hardware failure or malware.

### Scheduled Releases

If you ever wondered if there is a person running a build locally in the middle of the night and pushing it manually to the store to create a `NIGHTLY` release - there is no one like that. It's all scheduled tasks in CI.

Scheduled tasks in CI can handle time-consuming test suites, running them daily or weekly.

### CI Can Wrap Manual Steps into a Single Action

CI allows developers to automate repetitive tasks, such as creating a release. This includes building and signing apps, generating release notes, publishing to stores, or on servers.

## Why CI Skills Should Matter to You

Mastering CI/CD is an essential skill for developers looking to advance their careers. Employers increasingly value developers who can automate and optimize development processes. To fully harness the power of CI, it's beneficial to understand basic Linux operations and have some knowledge of bash scripting.

Trust me, nothing says "I'm a professional" more to a client than being able to build and deliver a fresh version of an application with a single click.

## Getting Started with GitHub Actions

To kickstart your CI/CD journey, I recommend starting with GitHub Actions. Begin with simple tasks, such as running tests after every push to the `develop` branch in your pet project (you do have one, don't you?). This initial setup will familiarize you with CI/CD concepts. GitHub Actions provides a wide range of ready-to-use actions created by other developers, which can be found in the [GitHub Marketplace](https://github.com/marketplace?type=actions).

This very blog is built using Github Actions. It utilizes a custom Docker image to generate PlantUML diagrams and perform other tasks. But that's a story for [another time](https://asvid.github.io/github-page-deployment).

## Conclusion

CI/CD is a game-changer for developers, offering automation, efficiency, and collaboration. By leveraging CI/CD tools and services like CircleCI, Bitrise, and Github Actions, junior developers can streamline their development workflows, ensure code quality, and automate tedious tasks. Embracing CI/CD skills not only enhances your development process but also boosts your professional growth in today's software development landscape. Start with Github Actions today, automate simple tasks, and unlock the full potential of CI/CD.
