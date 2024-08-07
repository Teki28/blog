---
layout: "../../layouts/BlogPostLayout.astro"
title: An AI web app with Supabase, Stripe and Nextjs
date: 2024-08-06
author: Teki
image: {
  src: "/images/post-9/job-lander-1.png",
  alt: "cover image",
}
description: As AI is really on fire recently, I felt that I need to make something. That's why this web app was created.
draft: false
category: Coding
---

# Job Lander - An AI web app with Supabase, Stripe and Nextjs

<p style="color: gray; font-size: 1.2rem; padding: 1rem">
  As AI is quite popular recently, I was wondering: How about making some App using AI and try to sell it, if it can go viral, I can quit my job and go hiking everyday. Then I picked one idea from my idea list: job lander - an AI tool to help students preparing their job interview. However, this project was dead without earning a buck when I am writing this, I just realized it's not a mature idea and can not make to PMF. So it's better to stop right here and leave an article to record how I made it and how I gave up.
</p>

### Overall Sturucture

Main components:

- Frontend: Nextjs, Tailwind css
- Backend: Supabase
- Payment Service: Stripe
- AI Service: Google Gemini
- Deployment: Vercel (<https://job-lander.vercel.app/> Hope it's still accessible)
(I used that starter kit to generate original code: <https://vercel.com/templates/next.js/subscription-starter>)
- github repo: <https://github.com/Teki28/job-lander>

![Architecture](/images/post-9/job-lander-design.png "Architecture")
<p style="color: gray; font-size: 1rem; text-align: center;">Architecture of job-lander</p>

### Why Nextjs?

As I need to provide server functions such as Authentication & Authorization, API call, etc. While I am too lazy to write a separate server, Nextjs naturally becomes a good choice as it provides out of box support of edge function, route handler and useful middleware function for auth.
Besides, it sounds cooler when I tell people I used Nextjs instead of vanilla create-react-app.
I also used this chance to re-read nextjs offcial docs. Among more than 30 advantages mentioned in offcial doc, I think what helped me most on my journey of building this wep app are:

- Easy to understand document
- out-of-box support of tailwind css and typescript: to be honest, I am not a big fan of typescript before as most of projects I worked on were just small projects, and for those projects usually I spent more time on debugging my typescript complaint than debugging the real bug. But I started using Java and worked on some big projects recently after starting work, I felt that type is really a necessary thing. Especially when you have a function with more than 1k lines without any comment & doc.
- concept of server & client component: I personnally like this design not only because it improves overall performance, but it also makes codebase cleaner and more algin to single responsibility principal, that we use server component to handle auth, data retrival and other backend logic, use client component to write unreadable javascript and css code.
- nice routing system: I used old page router several years ago, but luckily I forgot everything I learned at that time. So this time I didn't spend much effort on migrating and understanding concept of App router, which is a pretty nice and easy to understand design.
- nice integration with vercel: As nextjs and vercel are from same group of folks, they really made it smooth to onboard vercel with my nextjs app to the extent that I think even my grandma can deploy a nextjs to vercel. I may help her building a recipe website using nextjs and vercel later this year.

 You may notice when I talked about advantages of nextjs, I talked a lot on developer experience, instead of performance. That's because at least this time, as an indie hacker, I don't need worry much about performace issue, as I don't think there will be more than 3 people using my app at the same time. Besides, even if you need to build a app for large group of users, I think you should always put more effort on backend, like database and data structure. Just build a separate java/go service instead of spending too much time on client side cache, Img optimization blabla.

### Supabase

Supabase is a wrapper of postgresql database. On top of postgre sql database, it provides features including auth, edge function, out-of-box api and other advanced feature like realtime api and row level security. This kind of cloud hosting database is becoming trend in recent years, from firebase to supabase, by which users even don't need to build any backend service if their product only contains simple CRUD logic. What supabase suprises me is that it provides auth function, that I don't need to design any user and auth table, it can handle all auth logic like creating session, storing user infomation, etc. Besides, its row level security is also quite useful to build a safe data source. By using this row level security feature, we can directly define who can perform what kind of action(CRUD) on this row of data. A common use case will be user can only modify and delete their own data. Another big advantage of supabase is that it's open source and can be easily self-hosted, which open a gate for small team to build their product and service on top of it.
One thing to be improved I can think of is that: Not sure if it's possible to integrate other database other than postgresql, such as mongodb and other nosql database.

![supabase row level security](/images/post-9/supabase.png "supabase row level security")
<p style="color: gray; font-size: 1rem; text-align: center;">supabase row level security(Really useful!)</p>

### Stripe

Skip this part as I didn't get any subscriber till now, so I don't know if stripe is good or not. But one thing I heard from some random podcase is that it's really troublesome to really use stripe and earn money at early stage. As we need to have our own company and work on many legal and tax documents, although this is stripe's fault.

![Main page of job-lander](/images/post-9/job-lander-1.png "Main page of job-lander")
![Main page of job-lander](/images/post-9/job-lander-2.png "Main page of job-lander")
<p style="color: gray; font-size: 1rem; text-align: center;">Main page of job-lander</p>

### What I learned from this "failed" project

- It's easy to build something, but it's really hard to build something useful.
- It's easy to implement functions, but it's really hard to design and write UI.
- It's easy to design and write UI, but it's really hard to make a UI that is not ugly and hard to use.
