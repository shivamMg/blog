---
title: "//build/ 2023"
summary: "Microsoft Build 2023 and our team's work"
date: 2023-06-10
series: []
categories: ["Microsoft"]
tags: ["microsoft", "chatgpt", "ai"]
weight: 98
aliases: []
author: "Shivam Mamgain"
---

Been working heads down along with the team for the last couple of months. Our work is mostly around [Retrieval-Augmented Generation](https://arxiv.org/abs/2005.11401). Personally, I had a lot of learnings around Machine Learning, especially Large Language Models like GPT.

Part of our work was showcased at [Microsoft Build 2023](https://build.microsoft.com/en-US/sessions) ([Archives](https://build.microsoft.com/en-US/archives) for future readers). I've listed down sessions where our work was showcased or mentioned. I've also listed sessions that I really liked.

## Satya's Keynote - [Link](https://build.microsoft.com/en-US/sessions/49e81029-20f0-485b-b641-73b7f9622656?source=sessions)

AI dominated the keynote obviously. I recommend watching it to get a high-level idea of what's in the store in the coming few weeks. There have been multiple teams grinding to release several features. Our team's work is present on this slide (16:35 mark in the video):

![](/build-2023/teams-work.png)

Specifically these areas: Vector indexing, Retrieval Augmented Generation, Prompt workflows.

From the keynote, I particularly liked this slide that has historical references that give an idea of what our understanding was of future technology back then:

![](/build-2023/technology-history.png)

I've linked the first three:

- [As We May Think](http://worrydream.com/refs/Bush%20-%20As%20We%20May%20Think%20(Life%20Magazine%209-10-1945).pdf) by Vannevar Bush, July 1945
- [Man-Computer Symbiosis](https://groups.csail.mit.edu/medg/people/psz/Licklider.html) by J. C. R. Licklider, March 1960
- [The Mother of All Demos](https://www.youtube.com/watch?v=B6rKUf9DWRI) by Douglas Engelbart, December 1968

Another interesting slide was a graph around GDP growth with references to revolutionary technological advancements:

![](/build-2023/gdp-growth.png)

## Build and maintain your company Copilot with Azure ML and GPT-4 - [Link](https://build.microsoft.com/en-US/sessions/abe8f7cd-5aa1-4ee6-bb0d-f1aff17ba3c8?source=sessions)

Daniel Schneider and Seth Juarez demoed our team's work. At 38:00 mark, Daniel shows how to create a Vector Index in PromptFlow:

![](/build-2023/vector-index.png)

The full session covers not just our work but also of others around us in AzureML. Great demo session overall. There was some adhoc work I did for Daniel's Copilot demo. I created a websocket server for his Copilot, and a console client. He demoed it at 19:00 mark:

![](/build-2023/ws-console-app.png)

Looking at the console client Seth remarks: "If it was 1987 and this is the interface we give to our customers..." - _me crying_

## The era of the AI Copilot - [Link](https://build.microsoft.com/en-US/sessions/bb8f9d99-0c47-404f-8212-a85fffd3a59d?source=sessions)

Delivered by Kevin Scott (Microsoft CTO), it explains what a Copilot is with a nice demo ([code](https://github.com/microsoft/PodcastCopilot)) at 35:30 mark. Greg Brockman (OpenAI Co-founder) was invited on stage for a QnA. Kevin also mentioned Harrison Chase (Creator of LangChain) who was present in the audience. Nice.

## State of GPT - [Link](https://build.microsoft.com/en-US/sessions/db3f4859-cd30-4445-a0cd-553c3304f8e2?source=sessions)

My favorite session from the conference, delivered by Andrej Karpathy from OpenAI. I've been following his work for some time now. This session gives you an overview of the ecosystem that has come to exist around ChatGPT. It touches upon things like [Chain-of-Thought prompting](https://arxiv.org/abs/2201.11903), [AutoGPT](https://github.com/Significant-Gravitas/Auto-GPT), [LangChain](https://python.langchain.com/en/latest/index.html), etc. The best part is the explanation of how ChatGPT is trained. Deeply technical session. Loved it.

![](/build-2023/chatgpt-training.png)

## Building AI solutions with Semantic Kernel - [Link](https://build.microsoft.com/en-US/sessions/31e11443-70d3-4020-8c8c-0a654bccd233?source=sessions)

I've been using [LangChain](https://python.langchain.com/en/latest/index.html) to write LLM based applications. [Semantic Kernel](https://github.com/microsoft/semantic-kernel) by Microsoft is another alternative. A lot of overlaps between the two. Semantic Kernel was originally written in C# and has a Python package now.

## Building and using AI models responsibly - [Link](https://build.microsoft.com/en-US/sessions/70c6d334-0e4a-4235-ad57-92004b06d7e7?source=sessions)

Generative models spew all kinds of things. What goes in and comes out of these models will need to be adjusted, for instance, to filter out harmful responses by LLMs. ResponsibleAI works in this area. I feel this problem is big enough that we might see startups focusing on just that.

## End

Lot to learn.

---
