---
layout: post
title: Tech Retrospection on My 2017
---

The year 2017 has witnessed a lot of my growth - some were easy while some with struggles.
As the year comes to a close, I would like to drop some notes to myself about them before I forgot.

### Vue
If I were to choose only one keyword for my learning in 2017, I would definitely go with **frontend**.
And what I think as the most significant growth in me, among all the frontend stuff,
is having learned [Vue](https://vuejs.org/).

I have always wanted to pick up a proper frontend framework to expand my engineering capability.
3 years ago when I was still at my final year of university, I interned in Bubbly and dabbled into Angular 1.
It was the very first time I got to work on a frontend project that's really used by anyone at all.
In the end I was not able to ship the project as polished as we intended (and I was rejected of return offer),
but I realized how much frontend has evovled, and how much I was lacking in my web engineering experience.
The only construct I was able to harness in Angular 1 was the controller.
I didn't understand services stuff, as I didn't understand promise stuff.
Also the complication of concepts in custom directives has troubled me so much
that for once I thought I would never be able to comprehend the MVVM trend in frontend.

When I bumped into Vue while browsing tech blogs online, it was 2 years ago and it was still in 1.x.
Its simplicity caught me and I thought it would be an easier track to grok frontend engineering.
The idea was with me, on and off, but I was too involved in data processing projects in my work.
I've never reached anywhere more than "Hello World" level experiment.

Late last year I was summoned to shift my job scope to web related projects.
It was the time React stack received much attention, especially in our company's other departments.
Yet after comparison, I decided to go with Vue.
The consideration was mainly on our team's collaboration with design team.
We remotely work together with other design teams and they are often the producer of HTML and CSS.
Going in the direction of JSX and all the functional component design is way too aggressive for their background.
However back then Vue 2 was already in beta stage, with a few features deviated from Vue 1,
so I was not very keen to immediately dive into it.

Early this year when Vue 2.x was released proper - with a community posted enough tutorials for me to refer,
I thought it was finally the time for me to introduce it to the project at my workplace.
It was also right at this time, a slightly more complicated web project came to us.
The website requires quite a bit more interactions than our usual ones, and it has version extensions planned.
So I decided to give it a try by putting all the tool chain together for the project.

During the development, We did run into small collaboration issues here and there,
as none of us had prior experience constructing such a site using frontend tooling.
The places I stumbled upon were transition effects and state management.
The transition effects bit me mainly because I didn't have enough experience with CSS,
and our HTML/CSS slicer worked mainly with jQuery stuff alone previously.
State management side-tracked my development as I was not sure whether to learn Vuex or RxJS back then.
In the end, I dug through the transition document for Vue and found what I need,
and stick with the popular choice of using Vuex for state management.

And it was shipped, with much room for improvement, but in time.

Upon retrospection, it was a pretty stressful period, as I had to bear the risk of not being able to deliver.
Also even if we manage to finish in time, if I coded it with antipatterns all over the place,
I wouldn't be able to learn much from the project nor would I be able to share it to our team for adoption.
But thankfully, Vue is friendly enough for newbies like me,
and I finally did my first frontend project with tool chains and MVVM pattern this year.
This experience filled so much of my knowledge gap that a whole lot of program design paradigms I learned,
could now be put to very practical experiment in frontend projects.


### CSS (+ HTML)
With my JavaScript skill more or less on track to grow, the next item for me in frontend SPA development is the page content.
I had previously only very vague idea of CSS engineering practices.
Even I believe that it has a unique design pattern comparing to other general purpose programming languages,
the fact was that not many of us in the team addressed its importance sufficiently,
and I was not able to help with it due to my bad lack of experience.

After working on a few more projects, my understanding of the code quality in markup and styling was deepened.
Also by mid this year, I finished the book [_CSS Secret_](http://shop.oreilly.com/product/0636920031123.do) by Lea Verou,
which is a totally awesome book that led me into practical directions to improve code quality.
When I recommended it to one of my team members, she gave the same feedback as well.

![CSS Secret](https://covers.oreillystatic.com/images/0636920031123/lrg.jpg)

Moving forward to next year, our team would be working more web markup and styling within ourselves,
instead of collaborating remotely with different local design teams.
This should give us more control over the code quality and reduce issues when integrating sliced assets with JS code.


### Side Projects
The three most exciting projects I did (or started) on my own were

1. An OpenResty implementation of OAuth 2 authentication process
2. An OpenResty implementation of Docker socket event listener (on-going)
3. A tiny scripting language interpreter using JavaScript (on-going)

I have thought about implementing an OAuth 2 authentication process in OpenResty ever since I came across it early last year.
But my understanding of the OAuth 2 process was not clear enough to just do that;
plus my experience with OpenResty and Lua was still super shallow back then.
When I was on my annual leave this August, staying home with absolutely nothing better to do,
I thought I would just go ahead and nail it.
Then I worked on it for two days.
Surprisingly with relatively few lines of code, the process was completed, at least with Github's flavor.
I wrote a series of blogs in Chinese to document my thought process on Jianshu.
_(With Jianshu's decreasing popularity among programmers I'll perhaps translate them into English and port them over here.)_

After the completion of this simple OAuth 2 authentication implementation,
I was quite encouraged and wrote a middleware in Django to perform OAuth 2 authentication by
[request-oauthlib](https://github.com/requests/requests-oauthlib) for our team.
The middleware was ported into our team's cookiecutter template,
to unify the OAuth 2 authentication for Google and our own OAuth provider.
We used to have a self-implemented library based on [django-rest-framework](http://www.django-rest-framework.org/) for our own OAuth
and use [python-social-auth](https://github.com/python-social-auth/social-app-django-mongoengine) for Google's.
The new solution has much lighter dependencies and works uniformly in our project setup,
which I think is an improved developing experience XD

The Docker socket event listener idea was initiated with the fact that
our website deployment uses [docker-flow-proxy](https://github.com/vfarcic/docker-flow-proxy),
and it uses [HAProxy](http://www.haproxy.org/) which is not something I understand the configuration of XD

So I set off implementing one myself.

At first I implemented it with the wrapped API to HTTP provided by [lua-resty-http](https://github.com/pintsized/lua-resty-http),
but encountered a lot of issues, primarily with the timeout handling.
After consulting some senior developers of OpenResty in a QQ group, I changed my implementation strategy to use barebone socket API.
It turned out that with a bit of document exploration,
the functionality was done without any of the timeout handling issues I had previously;
the code that does the stuff was not as daunting as I thought it would be either.
I'll find time to refine it this coming year, with handling logic for the events we need in our environment.

The last project, a scripting language, is largely a JS rewrite of [an interpreter written in Go](https://github.com/mmyoji/go-monkey).
It's still on-going and I'm targeting to finish it before February.
I'll probably write a bit more on that when it's done.

I'm noting these projects here because they greatly increased my confidence
to implement something that's generic, not business logic centric,
and the domain knowledge they involve were not easy to grasp for me a while back.

-----

In closing, I'm expecting even more to happen in the year ahead,
as I would be able to drive more decision with increased seniority.
And I sure hope I can continue to grow my skill set further,
and build a working environment more fun and productive for my team members and myself :)