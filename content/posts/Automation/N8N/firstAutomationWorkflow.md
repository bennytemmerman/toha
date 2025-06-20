---
title: "My first automation workflow"
date: 2025-06-18T10:17:23+02:00
#hero: /images/posts/logrotate.svg
hero: /images/posts/n8n.jpg
description: Creating an AI agent to help learn a new language
theme: Toha
menu:
  sidebar:
    name: My first automation workflow
    identifier: myFirstAutomationWorkflow
    parent: cat-automation
    weight: 400
---

# My first steps into automation with AI agents using n8n

## Setting the stage: n8n in my homelab

As someone constantly tinkering in my **Proxmox-based homelab**, I recently decided to explore **workflow automation with AI agents** using [n8n](https://n8n.io/) — a powerful, self-hostable automation tool that connects anything to everything.

To get started, I spun up a new container and installed **n8n via Portainer**, running it in Docker. For anyone looking to do the same, here's the official n8n install guide for Docker and Portainer:

[Install n8n with Docker & Portainer](https://docs.n8n.io/hosting/installation/docker/)  
[Link to environment variables](https://docs.n8n.io/hosting/configuration/environment-variables/deployment/)

---

## Trial and error: The hands-on approach

Initially, I went in completely blind. I began dragging in random nodes, loosely configuring them, and hoped for some magic.

Spoiler: **it didn’t work**.

I kept hitting walls — nodes not connecting, credentials misbehaving, and the dreaded “nothing happened” moments. But each small failure taught me a bit more about how n8n thinks and how its workflows execute.

---

## 💡 The language learning idea

While experimenting, I had a lightbulb moment: why not use AI to help me learn a new language?

Here was the concept:

- **Schedule a daily trigger**
- **Use an AI agent to ask a chatmodel** (e.g., Google Gemini) for a new sentence in the target language
- **Include translation, pronunciation, grammar, and spelling tips**
- **Send the output to a Discord channel via webhook**

![n8n result](images/posts/n8nresult.png)

This felt like the perfect fusion of **AI**, **automation**, and a **personal need**.

---

## Turning to tutorials

Realizing I needed a stronger foundation, I dove into some n8n tutorials:

1. [Your First n8n Workflow](https://docs.n8n.io/try-it-out/tutorial-first-workflow/)  
   This tutorial helped me understand how to structure a simple workflow from trigger to output.

2. [Intro to Advanced AI Workflows](https://docs.n8n.io/advanced-ai/intro-tutorial/)  
   This one introduced the concept of using AI models within workflows — exactly what I needed.

After going through these, things started to click.

---

## The first major roadblock

I created my workflow: a **Schedule Trigger** connected to an **AI agent** and ending in a **Discord webhook**. It worked beautifully when I ran it **manually**, but… nothing happened when the scheduled time passed.

### Problem:
The `Schedule Trigger` didn’t fire on its own.

### Solution:
- My Docker container was running, but n8n wasn’t in **active/production mode**.
- I updated my Docker container's environment variables:
  - `N8N_EXECUTIONS_MODE=own`
  - `TZ=Europe/Brussels` (to match my timezone)
- I also **switched the trigger from interval to cron** (`45 9 * * *`) for reliability.
- Finally, I ensured the workflow was **activated and saved**.

After these changes, the scheduled execution worked like a charm.

---

## Early results & simplifying things

Initially, I tried to go **way too complex** — chaining advanced AI nodes, formatting logic, and conditionals. Eventually, I **scaled it back** to something simpler just to start getting data flowing and build confidence.

> The minimal setup: 
> Schedule Trigger → AI Agent → Discord Webhook

![](images/posts/AIAgent.gif)

An AI agent is not fully required as this could have been configured as an automation workflow, this was purely for testing purposes.
This project provided value and a great learning experience.

---

## Next steps & ideas

Here’s where I want to take this next:

- **Interactive Discord bot**: Respond to messages in a channel, maybe even answer grammar questions.
- **Clean-up logic**: Automatically delete older language messages to keep the channel fresh.
- **Add memory node**: Track which sentences have already been sent and avoid duplicates (or regenerate).
- **Logging to Google Sheets**: Keep a log of daily sentences and explanations for review.

---

## Final thoughts

Exploring automation with **n8n + AI** has been one of the most satisfying and productive rabbit holes I’ve gone down in my homelab journey. With a bit of trial and error, and help from the excellent n8n docs and community, I now have a working system that automates part of my language learning journey — and that’s just the beginning.

If you’re on the fence about diving into automation or AI integration, my advice? **Just start** — even the mistakes are great teachers.

---

Want to see more of my homelab experiments? Let me know — happy to keep sharing!
