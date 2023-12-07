```
author: @sleepyfox
title: ZDD - Zero Dependency Development
date: 22-May-2023
```

# ZDD - Zero Dependency Development

![Heaviest objects in the universe](https://d2908q01vomqb2.cloudfront.net/0716d9708d321ffb6a00818614779e779925365c/2021/10/14/node_modules-heaviest_object_in_universe.png)

Long build times, longer test suite run times, poor devEx, and now supply chain attacks, who isn't frustrated with huge dependency chains these days? The answer is simple: delete them all!

## TL, DR; 

Well, OK, it isn't quite that simple. #ZDD is like a 'Zero Defect policy', or #NoEstimates - it doesn't mean you don't ever make a mistake, or don't ever estimate anything. What it does mean is that you are mindful of your dependencies, and strive to minimise, and in an ideal world eliminate, dependencies, and dependencies of dependencies. We'll talk about the strategies for this later in the [How to](#how-to) section, but first I'd like to provide some background. For those in a hurry, please skip [ahead](#how-to).

## Current situation

Last week we learned that PyPi has [suspended new user accounts and projects|https://www.bleepingcomputer.com/news/security/pypi-temporarily-pauses-new-users-projects-amid-high-volume-of-malware/] due to the tsunami of malicious supply chain exploits and typo-squatting attempts:

> The volume of malicious users and malicious projects being created on the index in the past week has outpaced our ability to respond to it in a timely fashion

Supply chain vulnerabilities are nothing new, but the volume of attacks has been growing exponentially and all the major repositories PyPi, maven, RubyGems, npm are all suffering from the same onslaught. Given the potential value of a supply chain attack, this is not going to get better any time soon.

There have been some advances in this area, around such things as [SBOM|https://about.gitlab.com/blog/2022/10/25/the-ultimate-guide-to-sboms/], but the reality is that a complete SBOM for anything like e.g. React is at best years away. In the meanwhile we have very real issues.

We have been living in an age of blissful naivety, where we trust everything that the public Internet - and our module repositories, provides us. We have optimised for moving fast and breaking things. We have been leaving our front doors open, and inviting passing strangers into our houses.

Now the majority of people are good people, but the times when we could get away with this are certainly now over. Back in the old days we used to laugh at the banks like Goldman Sachs, who didn't allow their developers access to the Internet, and who wouldn't allow them to use any open source modules - if you wanted something it all had to be (re)written in-house.

## Create-blah-app case study

Whilst at a client last year we built a simple 'create-react-app'[1] style utility to provide template-based micro-service boilerplate creation. Our initial proof of concept used Yeoman-generator and had 38877 dependencies. Yeoman-generator itself has 541 dependencies and uses 36MB of space. This project required more than the usual number of npm clean installs, because of the very nature of a create-blah-app style boilerplate generator, which makes any delays in running the testing suite particularly irksome.

Once everything was working, I resolved to make the situation better and spent a couple of days pruning down the transitive dependency tree by the use of two main strategies:

* Removing a dependency completely
* Replacing a dependency with a smaller, lighter-weight alternative

In many cases I found it was possible to use the first strategy and replace a dependency, either because it was a 'nice-to-have' and we could just do without (colors.js) or because the entire library could be replaced with a few lines of code. This was not usually because of a 'left-pad' style library, but rather because an included dependency did many things, but we just required one specific, small thing. Yes, I know some people will view this as heresy, but sometimes copying and pasting a function from a library is better than including the whole library. Because:

> Everything in software is a trade-off

When replacing a dependency with a lighter-weight alternative, I found that using sites like [PackagePhobia|https://packagephobia.com/result?p=yeoman-generator] and [BundlePhobia|https://bundlephobia.com/package/yeoman-generator@5.9.0] were helpful. There's also [npmgraph|https://npmgraph.js.org/?q=yeoman-generator] and [Avanka|https://npm.anvaka.com/#/view/2d/yeoman-generator] that give a more visual output.

At the end of this I was able to reduce the size of the complete dependency chain from 38877 to just 52 modules. Apart from the enormous reduction in attack surface, this represented a very much faster build and test cycle, and a much nicer developer experience overall.

## How-to

As Tony Blair once said, it is time to get tough on dependencies, and tough on the causes of dependencies.

![Tough on dependencies, tough on the causes of dependencies](https://i.imgflip.com/7nmyno.jpg "Tough on Dependencies, tough on the causes of dependencies")

As mentioned in the case study above, the two key strategies for minimising, and ideally eliminating, dependencies are:

* Removing a dependency completely
* Replacing a dependency with a smaller, lighter-weight alternative

### Strategy 1: remove a dependency completely

The first of these requires a bit of mental fortitude, because at the moment the prevailing attitude can briefly be summarised as "Why did you write these five lines of code when you could have just imported Spring?"

Exaggerated? Only slightly. This is how we ended up with [AbstractSingletonProxyFactoryBean](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/aop/framework/AbstractSingletonProxyFactoryBean.html). I recognise that this is a ['two hard things' problem](https://gist.github.com/sleepyfox/b20579302ce05a9ac9f78c6003566989#foxs-9th-law-of-software-development-two-hard-problems) - and admit that the solution to this problem(s) is one that I could write an entire book about. Also, many other people have written books about, probably better than mine would be.

The reality is that in the vast majority of cases, writing five lines of code will be less buggy and more maintainable than including that dependency that you just googled from PyPi, Maven central, NPM or wherever. It won't need incessant bug fixes/security updates. A version bump will never reduce backwards compatibility, it will never clash with another dependency's dependent version. In actual fact, given my experience, the most likely outcome is that you or anyone else on your team will never need to touch it ever again. Even if your five lines are just a cut-and-paste from the library that you just deleted.

I know this sounds like heresy. I know this runs contrary to what all the software development 'thought leaders' have been saying about 'code reuse' for the last 40 years or so. But the truth is that a) they have been lying to you, and b) they didn't know what they were talking about.

Explanation: 'code reuse' is about **you** reusing **your own code**. Or, worst case scenario, using your own team's/org's code. Not downloading and 'reusing' some random schmuck's code from the Internet. 

Much has been said over the years about how it is madness to reinvent the wheel, and how it would be a huge waste of time to write your own web-server, when the Apache Project's own httpd does a perfectly good job. This is true, but it is true because httpd has 79 *current* major contributors, and hundreds (their description, not mine) of minor contributors. The initial release of httpd was in 1995, 28 years ago. So we can guesstimate around 1400 person/years of work. Obviously reinventing the wheel here is likely folly.

The same goes for many other major pieces of infrastructure, databases for example. Yet people do invent new databases every year, so obviously there is still a need. People do invent new web-servers, though not every year it seems (nginx?), so again, there is sometimes a justification even for these monumental fantastic beasts.

But we're not talking about writing your own web-server, or even proxy server (I did this, about 60 LOC), we're talking about writing a few dozen lines of code, max. And because everything in software is a trade-off, we're talking about doing this rather than reading many hundreds, thousands, or even hundreds of thousands of lines of library/framework code in order to understand the thing that we're importing.

### Strategy 2: replace a dependency with a lighter-weight alternative

This is perhaps the easier option from the team/org cultural perspective. Perhaps. When it comes to front-end JS frameworks, there's still a hell of a lot of inertia, though articles like [this one]() are beginning to make some headway. I recognise that certain things, more usually under the name 'framework', are more resistant to change than others because they insinuate their way into everything that you do. Rails is a good (or bad) example of this, Spring similarly. But let us not be defeatist.

For JavaScript tools like Bundlephobia and PackagePhobia above are good examples of tooling that is appealing to the people that want (or need) to reduce their dependency on dependencies, but you don't even need them. All you need to do to find the load of a new module is to this:

```bash
$ mkdir test
$ cd test
$ npm install <insert-module-name-here>
$ ls node_modules | wc -l
<total number of modules>
$ du -sh node_modules
<total disk space of modules>
```

This will tell you the total number of dependencies and the space they consume, for any given module. A similar method will work for your language of choice. This will enable you to run comparative tests on modules and their alternatives.

An alternative here is to use a library fragment - many libraries and frameworks release themselves in a series of split parts, described as 'modular' or 'plugin-based'. AWS-SDK, Jasmine and JQuery are popular examples. Any opportunity to remove the things that you don't need should be pursued. This is particularly a win-win, because you don't have to change anything about the calling code other than the import, and you don't need to convince anyone to use a different library or framework than they are already used to. Of course, it's not as satisfying as removing a dependency completely, but it's still an improvement. 

## Rebuttals

Perhaps I should just call this 'excuses'...

### "I can't possibly scrutinise tens of thousands (or more) of dependencies to ascertain code quality, bugginess, security attack surface..."

But you must! How else can you, as a professional, vouch for the quality and trustworthiness of the code that you are including in the company's app, service or other product? Saying "But we use it elsewhere in our org!" is just like your child saying "But Jo did it first!" and us as a parent responding "And if Jo jumped off a cliff, would you follow them?". Simply because someone else did something unwise, does not abrogate you of your own professional responsibility. 

Neither does the fact that it is 'hard', 'unreasonable' or any other adjective relieve you of your responsibility as a professional to vouch for the quality of *all* the code that you check in, including the dependencies.

### "I am not a security professional, how would I go about assessing the security risk of an app, library or framework with which I am unfamiliar?"

You don't have to, nor should you be attempting to. Your organisation either a) has a team of professionals to do this, or b) can contract with an external team to provide security consulting. If it hasn't been vetted, then it isn't approved, and you shouldn't be including it. This often leads onto:

### "But it is inconvenient/delays my project, and we are working to a deadline!"

You remember that you are the professional, right? This means communicating in a timely manner with your sponsor and stakeholders about what the anticipated risks and projected timelines are. It doesn't matter if you are a consultant, contractor or FTE, your professional obligations are the same.

As part of taking on a project, or product work, the first step is discovery, and just like legal discovery, this means understanding the current landscape and all of the liabilities that this includes.

### "I can just filter for packages with provenance"

OK, this is better, because at least you're trying. The problem at the moment is that most package managers don't support provenance e.g. Maven, or support it in only very limited forms e.g. NPM that only supports provenance from a GitHub Action build script - good luck if you use any other form of repository manager or build agent.

And _even if you are_ one of the lucky ones, all that provenance tells you is that the package that you are downloading was built, packaged and published by the person/org that you thought it was. It doesn't say anything about the quality, security or reliability of their code, neither does it say anything about their transitive dependencies.

At some point we will probably have fully provenance-compatible modules that have provenance from all of their transitive dependencies, which will be wonderful. It still won't prove anything about the non-functional requirements of any of that code, just that it hasn't been hijacked by some typo-squatting supply-chain attack.


[1] For reference: `create-react-app my-react-app` takes 1m34s on my dev laptop and installs 825 modules taking up 330Mb of space (create-react-app v5.0.1). People refer to React as a 'lightweight framework', I'm not sure I can reasonably agree, given these figures.
