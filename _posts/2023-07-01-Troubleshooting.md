---
layout: post
title:  "The art of debugging"
date:   2023-07-01 01:22:00
comments: True
categories: [Software, Debugging]
excerpt_separator: "<!--more-->"
---

One of my strengths at work is debugging/troubleshooting technical issues. I have been told that I have a knack for finding and fixing problems that others find hard to solve. Many people come to me for assistance when they encounter difficulties and I often give them effective solutions or workarounds that allow them to continue their work. Some of them attribute my success to a “magic” power that I have, which is based on my good “memory” (subject of many memes at work) and ability to recall details about systems that I worked on long ago. However, there is nothing magical about what I do. It is simply a result of practice and experience in dealing with technical challenges. There are a few core ideas that underpin these abilities, some of which I try to enumerate below

<!--more-->

### Attention to details
When debugging a problem, it is important to pay attention to the details. Sometimes, the details are hidden in the later lines of a logline or a stack trace. Some people only look at the first few lines and miss the valuable information that is there. This can lead to wrong conclusions or wasted time. Therefore, it is better to scan the entire logline or stack trace and look for clues that can help solve the problem.

### Holistic view of the problem
Quite the opposite of what we just discussed, but another skill that is useful for debugging is to have a holistic view of the problem. This means looking at the problem as a whole and not just focusing on a specific aspect of it. Sometimes, the problem may be caused by something that is not directly related to the code or the system that is being debugged. For example, I was debugging a problem related to a dirty write in a high-traffic application. The developer said that the logs showed that he was writing the record, but it was not visible later. When I looked at the logs for the whole session, I saw that the transaction was actually rolled back due to a concurrency issue. If I had only looked at the write operation, I would have missed the bigger picture and the root cause of the problem.

### Draw diagrams
A third skill that I find helpful for debugging is to be able to draw diagrams. When I talk to developers about a system or a problem, I like to sketch a quick diagram using tools like Excalidraw or Miro. This helps me communicate better and avoid misunderstandings. Sometimes, words are not enough to explain a complex system or a problem. A diagram can show how the components interact, how the data flows, what are the inputs and outputs, etc. I also think that drawing helps me remember more and I can always refer back to the diagram when someone asks me about a system or a problem.

### Taking notes
Taking notes is another skill that I practise whenever I learn about a system or a problem. I take notes in an application such as OneNote or Notepad++. They may not be very organised, but they help me create a mental map of the system or the problem. I write down some keywords, how the information flows between the systems, what are the assumptions, what are the questions, etc. This helps me gather information faster and keep track of what I have learned and what I need to learn more. Taking notes also helps me search/review what I have learned and recall it later.

### Basic knowledge of related technology
Another skill that I will mention is to have some basic knowledge of related technology. By this, I mean knowing more than superficial knowledge of some technologies that are relevant for debugging. For example, I don’t need to be an expert in Linux, but knowing how permissions work in Linux helped me solve a problem where a C# program failed in a Docker container due to permission issues. Likewise, knowing how HTTP, TCP and UDP protocols work helps me understand some of the network issues that may arise when debugging web applications or distributed systems. Having more than superficial knowledge of some technologies is really helpful in my opinion because it helps me solve problems faster and understand issues better

### Knowing when to stop
The final skill that I will talk about is knowing when to stop debugging. Sometimes, debugging can be very time-consuming and frustrating. It can also be tempting to keep digging deeper and deeper into the problem, hoping to find the ultimate solution. However, this may not be the best use of time and resources. Sometimes, it may be better to stop debugging and look for alternative solutions, such as applying a workaround, asking for help, or escalating the issue. Knowing when to stop debugging requires some judgement and experience. It also depends on the urgency and importance of the problem, the availability of other options, and the potential impact of the solution. Knowing when to stop debugging can help save time and energy, and avoid unnecessary stress.

When you look back, none of these are rocket science - all of them are common sense really. They are just simple and logical things that anyone can learn with practice. If I had to choose one skill that I would recommend you to focus on, it would be to draw good diagrams. This skill can help you communicate better, understand more, and advance your career.