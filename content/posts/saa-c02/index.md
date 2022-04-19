+++
title = "Passing AWS' Solutions Architect Associate Exam (SAA-C02)"
date = 2022-04-19
slug = "saa-c02"
+++

(📝 this is a draft. Expect things to move around until I make them stay still.)

Yesterday I sat for the Solutions Architect Associate exam from AWS, and I passed! Here are some notes from my experience.

## Training material

I used Adrian Cantrill's [course](https://learn.cantrill.io/p/aws-certified-solutions-architect-associate-saa-c02) and TutorialsDojo's [practice exams](https://portal.tutorialsdojo.com/courses/aws-certified-solutions-architect-associate-practice-exams/) as my main reference, complementing them with AWS exams, docs, whitepapers and architecture diagrams when needed.

## Study methodology

First phase: I went through all of Adrian's course, without skipping anything and doing all the labs (except for most of the ones that fell out of scope of AWS' free tier). This took me some time, mostly because I decided to get serious with getting the cert after some time just consuming the content semi-randomly during the week in a passive way.

Second phase: Read, fill (knowledge gaps), test, repeat. I scheduled the exam one month and a half ahead of time. In that period, I split the practice exams evenly so I didn't see any repeated questions each time I took one. After taking an exam I read the explanations for each answer, and I noted and researched every product that came up that I didn't know of (using AWS docs mostly). **That last thing is very important!**

Note: I didn't take any notes during the course because there were some made by a student which I thought would be good enough. Surprise surprise, it wasn't. The notes were good, but I needed the reinforcement one gets when taking their own notes. So after the first phase and throughout the second one I started to take them when I had to review something (which ended up covering almost everything).

## General tips and exam technique

For this I'll refer to Adrian's blog post: [Passing the AWS Certified Solutions Architect Associate (SAA-C02) Exam](https://cantrill.io/2020/05/24/Passing-the-AWS-certified-solutions-architect-associate-saa-c02-certification.html). I think it has everything I want to write and more.

The only thing I might add is: pay attention to the product lineup AWS has for big data/data analytics. I had some questions come up related to them apart from the "classics" and the things in Adrian's course (they showed up in the practice exams though). Namely: EMR, Glue, OpenSearch, MSK, QuickSight. At least try to understand what they do, how they are deployed/implemented and how to integrate them with other products. They might've been beta-test questions, but you never know. Also while we're at it: AppFlow, AWS Backup, Amazon MQ, AWS Batch. Well, everything that comes up in your learning process that you don't recognize, research it!

## Outcome and remarks

A lot of the time I caught myself thinking, "it's so cool that there's so much a small team or just one person can do with all these tools". Obviously this applies to cloud computing in general, and there's a cost/benefit analysis to be made in every situation, but still. I think you certainly gain some perspective.

Another thing that came to mind is, "I'm glad I had previous exposure to a lot of this". Mostly with networking, and in my case mostly acquired in formal education. I remember having to drop out of a cloud computing course I was allowed to take early on in college, because it went over my head at the time - I didn't even went through "Networking 101" before! Not only for lack of technical knowledge though, also the business perspective (like OPEX vs CAPEX and such). I think that dedicating some time to really understand the basics first gives compounding benefits along one's career. I'm even inclined to recommend taking a MOOC on computer networks alone if one doesn't have the knowledge prior to taking a Solutions Architect course. Maybe also some cryptography videos (symmetric vs asymmetric and so on). Or alternatively, consider the "fundamentals" section and videos from Adrian's course as the most important thing in the course if no previous exposure to those concepts.
