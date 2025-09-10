---
title: Personal Website
author: aidan
date: 2025-09-10 12:00:00 +0800
categories: [PersonalWebsite]
tags: [JavaScript, Medium]
description: Create your own personal website from scratch.
comments: false
pin: true
media_subpath: /assets/tutorials/personal-website
image: /aidanexample.png
---

## About the project

Welcome to creating your very own personal website. This is not a step by step tutorial to creating a personal website as that would defeat the purpose.

Your personal website needs to be unique and showcase you: what you have done and what you are going to do. This tutorial will help you though the entire process of creating your own personal website.

Here are some examples of personal websites: [sohumkapoor.com](https://sohumkapoor.com/), [arnavdev-two.vercel.app](https://arnavdev-two.vercel.app/), [aidankeighron.dev](https://www.aidankeighron.dev/)

## Designing the website

The first step in designing the website is figuring out the tech stack, or what languages and frameworks you should use.

The most popular language for web development is JavaScript or TypeScript so lets start there.

There are many different frameworks you can use to make a website in JavaScript:
- [**Next.JS**](https://nextjs.org/)
- [React](https://react.dev/)
- [Svelte](https://svelte.dev/)
- [Express](https://expressjs.com/)
- [Angular](https://angular.dev/)
- [Vue](https://vuejs.org/)
- [Vanilla JS](https://www.geeksforgeeks.org/javascript/what-is-vanilla-javascript/)

We recommend Next.JS or React as they are both simple and easy to use while allowing for a lot of creativity and customization.

If you want to use another language there are a few other options but most of these just serve HTML and CSS.
- [Java (JavaApplets)](https://www.oracle.com/java/technologies/applets.html)
- [PHP](https://www.php.net/)
- [Ruby on Rails](https://rubyonrails.org/)
- [Python (Django)](https://www.djangoproject.com/)
- [Go](https://go.dev/doc/articles/wiki/)
- [C#](https://learn.microsoft.com/en-us/visualstudio/get-started/csharp/tutorial-aspnet-core?view=vs-2022)
- [Perl](https://www.perl.org/)

> If you are using React or Next.JS you will need to decide on if you should use [Tailwind](https://tailwindcss.com/) or not. I would recommend Tailwind if you have worked with CSS before. It is a great option and makes styling much easier. If you are not comfortable with CSS or you haven't used it much I would stick with CSS.
{: .prompt-info }

Now that we have the technologies figured out we can talk about the UI design of the website. If you want to you can jump straight into development and figure out the UI as you go but if you want to plan it out you can use [figma](https://www.figma.com/).

Figma will let you design the entire website UI and then you can use it for reference when turning that UI into code.

[Here is an example of a figma design of a website](https://www.figma.com/design/I1EEwTG9J9TLDD0T2l1nd7/Personal-Website?node-id=0-1&p=f&t=SsMfxfyIOclclnoG-0) and [here is the resulting website](https://www.aidankeighron.dev/).

You don't need to design the entire website in figma, you just want a starting point.

## Adding something special

You want your personal website to stand out and be unique. You don't want your website to look like every other website out there.

The way to do this is to find a unique visual style or a unique feature/functionality. 

If you have an artistic flair a unique visual style can work best: adding animations, custom font, and specific colors and formatting can all come together to create a distinct website.

The other option is to create a unique feature/functionality. This could be anything you can think of: maybe you embed an entire project you worked on into the website; [Or make your entire personal website a driving simulator](https://bruno-simon.com/); Or create a match-3 game where you match programming languages and when you match 3 it show a project you made in that language.

The driving simulator was build with [threejs](https://threejs.org/), if you are using React you can use [react-three-fiber](https://r3f.docs.pmnd.rs/).

## Hosting and Domain name

### Hosting

Once you are done with your personal website how do you host it and show it to the world?

There are a few options for hosting:
- [Vercel](https://vercel.com/)
- [GCP](https://cloud.google.com/run/docs/quickstarts/frameworks/deploy-nextjs-service)
- [AWS](https://docs.aws.amazon.com/amplify/latest/userguide/getting-started-next.html)

We would recommend Vercel as it has a very generous free tier (I have 4+ website hosted and I am only using 10% of my limit) and it does not require and payment or credit card info to setup. It also works very well with NextJS.

All you need to do is go to [Vercel](https://vercel.com/new), create an account, connect it to GitHub and walk through their tutorial step by step and at the end you will have a live website you can send to all of your friends.

### Domain Name

But the default website will have a domain name that looks something like `<Project Name>.vercel.app`.

If you already have a custom domain name you can [add it to your Vercel website easily](https://vercel.com/docs/domains/working-with-domains/add-a-domain).

If you don't have a domain there are many options for purchasing one:
- [Porkbun](https://porkbun.com/)
- [GoDaddy](https://www.godaddy.com/)
- [Namecheap](https://www.namecheap.com/)

Costs are usually $10-$15 a YEAR, yes a year they are not very expensive.

You are also not limited to a `.com` domain, you can get a:
- .com
- .dev
- .tech
- .expert
- .boats
- .today
- .移动
- And many many more

Now you have everything you need to go ahead and create your own personal website!