---
title: GitHub-hosted Static Pages - is it worth it?
date: 2022-09-11
categories: [Blogging]
tags: [web development, blogging, software philosophy]
---

## Motivation for a github.io page


![github-screenshot](/assets/img/posts/github-screenshot.png){:class="img-responsive"}

[Github](https://www.github.com/) has a fairly strong positive reputation (as of this writing 🙂) 

They also have a [feature](https://pages.github.com/) where they will host a static website for you for free based on the contents of a github repository. By convention, most people use this as a personal blog space, and specifically technical blogs (I think this is a self-selecting mechanism I'll talk about later). So, using this system, you can have your own site associated with your github account. Nifty.

Throughout the years, I've had accounts on dozens of blogging platforms, along with setting up my own self-hosted blogs.

I've never gone with static blog generation in the way that github recommends though, so I thought I'd give it a whirl.

Here are some of the pros and cons I found in that process.

***

## Pros

### ✅ Free hosting and infrastructure maintenance

No hosting costs or subscription costs, big plus. But this extends beyond simple hosting costs and also covers maintenance costs. You don't have to update the underlying server, ensure it's secure, do any migrations[^3] or anything. No renewing SSL certificates, renewing domain names, etc. Theoretically, your content will be fine, secure, and accessible for as long as github offers this service. 

As someone who has unfortunately left Wordpress installations idle for years, I can say this point shouldn't be overlooked.

### ✅ Full Content control
You control the code that's deployed in its final form; the platform doesn't add advertising banners without you knowing or otherwise break up or reflow your content in ways you didn't anticipate.

### ✅ Full presentation control

Similarly, since the site has been auto-generated already, you can edit the generated HTML files after-the-fact and modify/inject whatever you want, as opposed to having to follow the presentation conventions of another platform, which might not allow certain style changes or code injection. They won't suddenly have a high-level meeting design meeting where they decide to change the color schemes around on you or anything.

### ✅ More Secure than a self-hosted, interpreted blog

Your typical blog (something like [wordpress](https://wordpress.com)) has all of the blog content inside database objects, and then runs it through an interpreter (like PHP in this case) to render the site. This adds an extra layer of complexity in the 'serving up a webpage' process, and this is the most vulnerable part, because typically the process rendering the site needs access to lots of other sensitive systems (minimally a database). While it's possible to set these sites up in a way where you effectively get a static site out of it (more on that later), you still usually have an administrative control center that is also easily accessible from the web. Securing all of the above and keeping it secure through migrations has a pretty large overhead, one that usually is beyond the scope of someone just trying to write something and put it up online.

### ✅ Nerdy satisfaction / IKEA effect

This process isn't technology-blind -- you're moving text files around and doing git commits to make it all work. Even if it's not optimal, there's a nerdy satisfaction to be had from it. You've wired a bunch of things together to make something happen. That adds something extra for some folks (such as myself 🙂)

***

## Cons

### ⛔️ Jekyll's moderate complexity

It seems very straightforward... you write a markdown file, run it through a templating engine, and you get a fully-rendered HTML page ready for publishing. You could even save the versions to Git so you can push it directly from a git repo.

This was the core promise [Jekyll](https://jekyllrb.com/) was created to fulfill [^1], coincidentally created by the cofounder of GitHub.

It's mainly for this reason that Jekyll is the engine that Github supports **the most**. By that, I mean there are tight [integrations](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/about-github-pages-and-jekyll) around it.

This means, in theory, you can push a raw markdown file to your Jekyll repo, run an automated build process, have your site generated for you and updated immediately. In theory. 

In practice, there's a little more tweaking involved. Minimally, you have to learn some of the basics of Ruby, Jekyll, and the GitHub build/actions system. There are some decent step-by-step tutorials out there, but I saw my set-up deviating in places enough that I had to do some investigating.

Typically, this meant just googling things like "how to add a missing gem dependency", but these little learning tasks add up. Not a lot, but it ends up being more than one would expect when 'just' applying a template to a text file and 'just' pushing that result, especially if Ruby and Git actions are new to you.

### ⛔️ The database lives in your head

As mentioned above, in a typical blogging solution, you have an admin interface with a [WYSIWYG](https://en.wikipedia.org/wiki/WYSIWYG) editor. That editor usually has convenient lists of meta-data fields built in, with format-checking and auto-completions where applicable. For instance, if you're able to set a language on your post, the editor will usually have a 'laguage' field with the languages you've declared your blog to have elsewhere. If a 'date' field is mandatory, the editor will make the 'date' field light up red if you forget it. 

In one of these static-generated systems, the schema for these fields lives in your head. You have to remember which fields are mandatory, what options they can take, how you set them, etc. This isn't a large set of fields, but can be cumbersome, especially with things like tags. You probably won't remember if you've tagged your posts with "programming" or "programming - c" or "c programming" later without having a set of existing tags available to you. Here, you just have to remember these things.

This goes for site-wide admin settings too. You have config files and you have to read lots of docs to know what settings they can take, like any other *nix utility.

### ⛔️ You kind of do have to use Jekyll

As mentioned above the learning curve is mostly due to the fact that Jekyll and github actions seem specially designed to work together, but strictly speaking you don't *have* to use Jekyll.

In fact, you could just write raw HTML and post it, and it would appear, published and ready to go. Of course, writing pure HTML would be a pain to do constantly. It would probably be better to at least have a template for the parts that don't change.

This is about the level of functionality [Pelican](https://getpelican.com/) provides. It was actually quite appealing to me personally because it uses Python and Jinja, technologies that I'm already very familiar with. It also had a very minimal amount of meta-data and configuration associated with it. I ran it, and voilà, a site ready to publish!

Except, the default themes aren't necessarily responsive. Sure, you want to put something out, but you want it to also look decent on mobile. And the default themes don't necessarily integrate well with github's build system without tweaks. And what if you want to put a diagram in?

Unless you literally just want to give a URL to minimally stylized text, you kind of have to use Jekyll because: 

* it seems to have the best community support in terms of styling, typography, responsiveness and other 'modern' design features that even the most casual readers will take for granted
* its special relationship with github means it receives first-class support for new features that github rolls out (like [giscus](https://giscus.app/))
* enough third-party extensions for common use-cases that you might not even realize you want until you start blogging (like diagrams)

### ⛔️ You'll need external tools for subscription-style features

There's some platonic ideal world where a blogger can simply produce content and that content appears in front of the people who would enjoy it most. Unfortunately, this isn't how content distribution works.

A modern blogger has to put lots of thought into discoverability. This means SEO, managing subscribers, having publishing schedules to social media, among many other marketing-like activities.

Most blogging platforms do have some amount of awareness of these factors baked in to them. Full-featured self-hosted solutions (like wordpress or [ghost](https://ghost.org/)) have tools and ecosystems around collecting subscribers, publishing schedules, ways to optimize content etc. Blogging sites, such as Medium and Substack, also have large parts of their platform dedicated to building audiences and monetizing. 

A static site, by nature, will be limited on how much of these tools it can provide. If you want to get into blogging more 'seriously', you will need to have external tools dedicated to managing the above

***

## Ideal user and use-case

So, who should use github's static blogging and when? 

Here's a checklist:

- [ ] You're tech-savvy (can live in the command-line)
- [ ] You enjoy writing markdown
- [ ] You just have some information you would like to post online "without fuss"
- [ ] You have some external tooling and way of distributing your content OR you don't care about distributing your content and you're building this 'for fun'
- [ ] You don't want to deal with maintaining servers

By the nature of these use-cases, this feature will most likely be used as an additional place to document technical information, probably where a person would like to give personal/editorial commentary that might not be appropriate in technical documentation. I think that's what we'll see this space used for.

Hope this was helpful!

*On a personal note, I'm going to experiment around with this for a while. I don't know how much I'll post or maintain it, but I'm confident that any technical tidbits I want to share will stay up and remain maintained well into the future, so I'll put content here that fits that need.*




## Notes:


[^1]: [Blogging like a hacker (archive)](https://archive.ph/1tjTP)
[^3]: Unless you use the built-in build process code






