---
title: "Review of the CRTO Exam by ZeroPoint Security: My Journey and Insights"
date: 2024-07-31 11:33:00 -500
categories:
  - Exam-Reviews
tags:
  - exams
  - reviews
  - redteaming
---

![](/assets/img/CRTO-Review/image.png)


# Introduction

I recently joined a new company and set my sights on moving into the red teaming side of cybersecurity. To achieve this goal, I completed Zero-Point Security's Red Team Operator (CRTO) course. After about 3 months of intensive study and hands-on practice. **Today, I'm thrilled to share my experience and insights about the CRTO course and exam, capturing all 8 flags**.

![](/assets/img/CRTO-Review/badge.png)

# Background Story

### About CRTO Certification

The Certified Red Team Operator (CRTO) certification by ZeroPoint Security is a highly regarded credential in the cybersecurity field. It focuses on simulating real-world attacks, covering the entire attack lifecycle from initial compromise to full domain takeover.

### Why I Chose This Certification 

Over the past year, I’ve poured countless hours into experimenting with Active Directory exploitation and various open-source C2 frameworks in my lab. My passion for advanced penetration testing had me eyeing several certifications, including Zero-Point’s CRTO, OSCP, CPTS, and others.

Ultimately, the CRTO course stood out because it covers the entire attack lifecycle, from initial compromise to full domain takeover. This real-world focus was exactly what I needed to level up my red teaming skills and further my career aspirations.

Now, I'm excited to continue my journey by pursuing the Certified Red Team Lead (CRTL) certification.
# Course Structure and Content 

**Course Overview:** The CRTO course is structured into several modules, each focusing on different aspects of red teaming. It covers topics such as initial access, lateral movement, privilege escalation, and domain dominance and AV Evasion. The course also includes extensive hands-on labs that allow you to apply what you’ve learned in a simulated environment.

**Hands-On Labs:** One of the highlights of the CRTO course is its practical, hands-on labs from snap labs. These labs are designed to simulate real-world scenarios, providing an immersive learning experience. I found these labs incredibly helpful in understanding and applying the theoretical concepts taught in the course.

**Resources Provided:** ZeroPoint Security provides a wealth of resources, including detailed study guides, instructional videos, and lab manuals. These resources are invaluable for reinforcing the concepts and techniques covered in the course. Additionally if you enroll for this certification, **Lifetime access** to the online course documentation, which sections are constantly being updated.

![](/assets/img/CRTO-Review/course-curriculam-1.png) 

To understand more about the course, you can visit the official page of [zeropointsecurity](https://training.zeropointsecurity.co.uk/courses/red-team-ops).

**Preparation Strategy:** I dedicated about 1 months of active preparation while 3 month of passive preparing for the CRTO exam. My strategy involved a combination of reviewing the course materials, practicing in the labs, and experimenting with cobalt strike C2 frameworks and Active Directory exploitation techniques in my own lab setup.

![](/assets/img/CRTO-Review/Notes-2.png)

# Pricing

In Comparison to lot of other certifications and red teams trainings, CRTO is very affordable. You can purchase the whole course and just the exam. Another thing to mention of the course is you have lifetime access to the course material and its future updates.

For more detail about the course, you can visit [here](https://training.zeropointsecurity.co.uk/courses/red-team-ops).

![](/assets/img/CRTO-Review/price.png)

# The Exam Day Adventure:

It all began with a range of mixed emotions. Anxious, excited, nervous, doubtful .......! but I finally scheduled my exam for July 6th at 11:30 AM IST.

My strategy was simple: 12 hours a day, over 4 days, wrapping up by July 9th.

With my fully charged laptop, my obsidian notes open, and a cup of coffee by my side, I was ready to dive in.
### Exam  begin.....

So, the exam kicks off, and we're handed a threat profile for the APT we need to imitate. I had to whip up a custom malleable C2 profile to mimic the network traffic of this specific APT group. Thankfully, I had practiced this a ton in my lab, so I just tweaked my existing profile.

Next up was setting up the initial Cobalt Strike infrastructure—payloads, listeners, AMSI bypass scripts, you name it. The trickiest part? AV evasion. Every time I generated a payload, it got flagged. I fired up `Ghidra`, reverse-engineered the payload, and found the strings that were getting caught. Tweaked the code, and bam—`ThreatCheck` showed a clean payload. But then it wouldn't connect to my listener. Great.

After some frantic Googling and note-checking, I finally got the payload to bypass AV and connect back. Three hours later, we were cooking. The first flag popped up at 2:42 PM, and the next two came quickly after. Smooth sailing, right? Not so fast.

The fourth flag had me stumped. I took a break, thinking three flags were good enough for the day. After a nice tea break in Bangalore’s rainy weather and dinner, I jumped back in. At almost 10 PM, I cracked the fourth flag.

The fifth flag required jumping to a different AD forest. My notes made it sound easy, but nothing worked. After another break and a different approach, I snagged the fifth flag at 12:55 AM.

With only one more flag needed to pass, I was pumped. Soon enough, the sixth flag was mine. I had passed! But, being the night owl I am, I couldn’t resist going for more.

Guess what? I got the eighth flag before the seventh, and then snagged the seventh in under a minute. By 3 AM, I had all the flags and had aced the exam.

And that’s how I conquered the CRTO exam—through persistence, breaks, and a bit of Bangalore rain!

![](/assets/img/CRTO-Review/points.png)
![](/assets/img/CRTO-Review/machines.png)


# Takeaways and Study Tips

Here are some golden Tips:

1. **Details Matter:** Pay attention to every detail in the course. You never know when it will be crucial during the exam. read the section carefully considering all OPSEC point.
2. **Test Your Arsenal:** Before the exam, test everything in your lab to avoid wasting time on errors during the exam.
3. **Create a Cheat Sheet and Comprehensive Notes:** Have a reference guide ready for quick look-ups.
4. **Handle Restarts:** When you restart, sessions will end. To minimize hassle, consider installing a backdoor or dumping credentials to retain persistence.
5. **Take Breaks:** If something isn't working, take a brief rest. Even computers need a break sometimes.
6. **Backup Beacons:** Keep a backup beacon. If one dies, you can use another. Organize your beacons and listeners well; it can get messy quickly.
7. **Stay Nourished:** Take breaks, and have some food and drinks. 48 hours is a long time.
8. **Document Everything:** Pause to document your steps and commands. This will help if you lose access or need to reboot labs.
9.  **Everything is in Course** : Do not try to do something out of different, all the required tool and knowledge is in the labs and course itself.
10. **practice practice practice :** solve all the module at least twice and understand why attack works . go for an extra mile challenge.

# Closing Thoughts

Finally, earning this certificate was an excellent decision to enhance my skills in red teaming and showcase my technical and critical thinking abilities. I am grateful for the support of my colleagues, friends, and everyone who helped me along the way. The course material was exceptional, and I'll definitely be using it as a reference for future red teaming engagements.

If you want to gain real red teaming skills and get hands-on with Cobalt Strike, I highly recommend this course.










