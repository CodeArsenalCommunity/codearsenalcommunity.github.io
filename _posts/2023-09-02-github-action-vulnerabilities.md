---
layout: post
title: "Securing GitHub Actions: Understanding Unpinnable Actions"
author: rohitcoder
categories: [GitHub Actions, Security]
tags: [GitHub, CI/CD, Security, Unpinnable Actions]
image: https://i.imgur.com/xBNyQ60.png
featured: false
hidden: false
---

In this article, we'll explore the security of GitHub Actions. These are like digital helpers for automating tasks in software creation. They're great, but they come with security challenges. We'll focus on a specific problem called "unpinnable actions" and show you how to keep your workflows safe.

## Why Pinning Matters

GitHub Actions help automate tasks like testing and deploying software. When you use actions made by others, it's crucial to **"pin" them to a specific version**. This means you lock them to a particular edition to prevent sneaky attacks.

Pinning makes sure you always use the same version. This is a security recommendation to make sure your work is safe.

![Figure 1](https://i.imgur.com/UNU2uPT.png)
*Figure 1: A safe, pinned version of a third-party action in a GitHub Actions workflow.*

This seems like a great way to stay safe from attacks. But, as you'll see, it's not perfect.

## How Attackers Exploit Unpinnable Actions

Let's talk about actions that can introduce harmful code to your projects, even if you've "pinned" them. We call these "unpinnable actions."

To understand this, let's look at an example:

![Figure 2](https://i.imgur.com/4P59cXc.png)
*Figure 2: An example workflow using the pyupio/safety action, pinned to a specific version.*

Imagine a workflow called "CI" that uses a third-party action called "pyupio/safety." It's pinned to a specific version, so you'd expect it to always use the same code.

But when you look at the action's code, it pulls the latest version of a Docker image named "pyupio/safety-v2-beta" from the internet. This image becomes the environment where the workflow runs.

![Figure 3](https://i.imgur.com/uVYdKpp.png)
*Figure 3: The action.yaml file of pyupio/safety pulls a changing Docker image with the 'latest' tag.*

Docker images use a "pinning" concept based on a unique code. The action gets the "latest" tag, which can change. Even though you "pinned" the action's code, the environment it runs in can look different each time.

![Figure 4](https://i.imgur.com/0RzMWjc.png)
*Figure 4: An attacker can trick action pinning by using a harmful Docker image with the "latest" tag.*

If attackers take control of the "pyupio/safety-v2-beta" image, they can put bad code in it. This code will run in your workflow, even though you "pinned" the action's code. That's how they can attack your project.

## Types of Unpinnable Actions

Let's look at three types of actions - Docker container, composite, and JavaScript - and how attackers can bypass action pinning.

### Docker Container Actions

Docker containers are like customized environments for actions. They're useful when you need specific tools or setups. These containers can be fetched from the internet or built on the spot.

![Figure 5](https://i.imgur.com/AcOy6iB.png)
*Figure 5: The Dockerfile of an action with unverified dependencies.*

As you can see in Figure 5, the process of building these containers can sometimes install software without checking if it's safe. This goes against the idea of pinning actions for security.

### Composite Actions

Composite actions are like shortcuts for simple tasks. They let you write code directly in your project files. But if you're not careful, they can break action pinning. Look at this example:

![Figure 6](https://i.imgur.com/m2R3TDk.png)
*Figure 6: A composite action that uses another action without pinning it to a specific version.*

This action uses another third-party action called "BrianPugh/install-micropython," but it doesn't pin it to a specific version. If attackers mess with "BrianPugh/install-micropython," your project becomes vulnerable.

There's another example with an action.yaml file that uses an NPM package without locking its version. Attackers can change this package, and your project can get hurt.

![Figure 7](https://i.imgur.com/Q8PebV6.png)
*Figure 7: A composite action that uses an unverified NPM package.*

### JavaScript Actions

JavaScript actions (sometimes TypeScript) are harder to break with pinning, but they can still fetch things from the internet. Here's an example:

![Figure 8](https://i.imgur.com/X4nnEfa.png)
*Figure 8: A JavaScript action that downloads something from the internet without checking it.*

Compared to other actions, JavaScript actions don't install new software, but they can get things from the internet. This makes them hard to pin for security.

## Unpinnable Actions in the GitHub World

GitHub Actions are popular in open-source and private projects. Many people use them, believing they're safe. To find out, we looked at the GitHub Marketplace. We found that 32% of the top-starred actions there were "unpinnable." This means they could still be risky even if you pinned them.

![Figure 9](https://i.imgur.com/HQJQaVb.png)
*Figure 9: 32% of the top-starred actions are unpinnable.*

We also studied open-source projects on GitHub. Out of 2,000 projects with about 6,000 workflows, 67% were using "unpinnable" actions. So, even in open-source projects, the risk is high.

![Figure 10](https://i.imgur.com/WefeELB.png)
*Figure 10: 67% of open-source projects use unpinnable actions.*

## How to Stay Safe

Now that you know about "unpinnable" actions, let's talk about staying safe.

### Prevention

1. **Check and Lock Actions**: Regularly review actions you use, especially third-party ones. Make sure they're locked to a specific version.

2. **Private Action Registry**: If possible, have your own action store. This gives you more control over action versions and reduces risks.

### Detection

1. **Keep an Eye**: Watch your GitHub Actions for strange behavior or changes in code.

2. **Use Security Scanners**: Add tools to your pipeline to spot problems and odd behavior.

### Response

1. **Isolate and Investigate**: If you find something fishy, check it out and find the source.

2. **Update and Fix**: If you confirm an issue, update affected actions and fix any problems.

## Conclusion

Pinning actions is a smart move, but it's not perfect. Unpinnable actions can still put your project at risk. By following our advice on prevention, detection, and response, you can make GitHub Actions safer and avoid supply chain attacks.

---

*Original article by Yaron Avital. For more details, you can read the [original article here](https://www.paloaltonetworks.com/blog/prisma-cloud/unpinnable-actions-github-security/). Adapted and simplified by Rohit kumar.*
