+++
date = 2025-11-29T06:30:00Z
disable_comments = false
tags = ["yantra", "workflow", "golang", "automation", "graph", "river queue", "rube goldberg machine"]
title = "Yantra v0.1"
type = ""
draft = false
+++
![Yantra workflow screenshot](/demo-workflow.jpg)

# Journey so far and v0.1 alpha announcement
It's been close to 4 weeks since I last [wrote](/posts/yantra-workflow-automation) about Yantra. Within a week I wanted to wrap up the project, clean it up and then make the repo public. In a classic case of chasing the next interesting project to do, I got distracted with electronics and began 3 separate projects (Will write about it in the future). I was also apprehensive about sharing the code as I kept finding edge cases that I wanted to fix before release. It was an endless cycle of fixing and testing. So I finally had to take a call and release it in whatever state it is in.

So I'm happy to announce [Yantra v0.1](https://github.com/patali/yantra). I have added a couple of working examples that you can easily import to test out various workflows. The example workflows should give users a decent understanding on how to use inputs and outputs and wire up everything. 
![Yantra workflow examples](/yantra-examples.jpg)

Apart from the examples I've also added a new executor node called **Sleep Until**. Thanks to the existing cron based workflow architecture, it was quite easy to introduce indefinite suspension of a workflow (Which seems to be all the rage lately). So now you can suspend a workflow execution midway for couple of hours, days, or months or make it execute on a specific date in the future. That was super fun to implement.

Github link: [https://github.com/patali/yantra](https://github.com/patali/yantra)

# Disclaimer
- The software has janky parts, especially in relation to Node's I/O. 
- Yantra was written with the help of LLMs so be wary of using it in production environment if you decide to do so
- The workflows are built for my use cases, so if you find any edge cases not working please let me know.

---

*Yantra is a workflow automation tool built with Go and Vue.js. It features visual workflow building, checkpoint-based resumption, the transaction outbox pattern for reliability, and horizontal scaling with stateless workers.*