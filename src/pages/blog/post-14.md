---
layout: "../../layouts/BlogPostLayout.astro"
title: Claude Code Note - How it works and How to make it works better
date: 2026-03-15
author: Teki
image: {
  src: "/images/post-14/cover.png",
  alt: "cover image",
}
description: What I have learned when I started Claude Code from scratch - lessons and pitfalls
draft: false
category: Coding
---

## LLM and Agent - What are they and what's the relation

### What's LLM?

In short, LLM (Large Language Model) is a huge function with tons of parameters and dimensions that aims to predict what the next output is for a given user input.

The key difference from a pure function is that LLM is based on probability and is a black-box inside. Unlike a pure function — where the same input always returns the same output — LLM will give you roughly similar outputs for the same input, but never exactly the same result. This is one of the major limitations preventing it from being used in scenarios where certainty is everything, such as surgery. It would be terrifying if a doctor could not assure doing the same thing every time.

![What's LLM](/images/post-14/whatsllm.png "What's LLM")
<p style="color: gray; font-size: 1rem; text-align: center;">What's LLM</p>

### How LLM is created?

LLM originates from deep learning and neural networks, which is not something new. It has actually been around for years but didn't seem powerful enough compared to other techniques, such as traditional machine learning in the early days.

It only evolved in the last couple of years, driven by the growth of computation power.

Let's take facial recognition as an example to dive deeper. There are two approaches: one is to come up with a powerful algorithm that calculates the similarity between two faces using their features (machine learning); the other is to simply memorize all the faces in the world (LLM). That's why data and computation power are so important for LLM.

![How LLM is created](/images/post-14/howllm.png "How LLM is created")
<p style="color: gray; font-size: 1rem; text-align: center;">How LLM is created</p>

### Why do we need Agent?

LLM is powerful, but it is not perfect. It has its own limitations — and agent is a solution to address them. So before we dive into agents, let's first look at the limitations of LLM.

![Limitations of LLM](/images/post-14/llmbad.png "Limitations of LLM")
<p style="color: gray; font-size: 1rem; text-align: center;">Limitations of LLM</p>

#### Knowledge cut-off

It only knows what it has been trained on, which means it has no awareness of events beyond its training data. For every LLM there is a knowledge cut-off date — for example, if the cut-off is 2025-09, it will have no idea what's happening right now in the Middle East. If you ask about anything beyond that, LLM will make up something based on its training data. This is called **hallucination**.

#### Hallucination

As mentioned above, LLM makes things up when it doesn't know the answer. This is a major problem and can be very dangerous in high-stakes scenarios like medical diagnosis — wrong information can lead to wrong treatment. This limitation stems from the probabilistic nature of LLM itself.

#### Lack of reasoning ability

LLM is good at generating text, but it struggles with precise reasoning. For example, if you ask LLM `1+1`, it will probably answer `2` because this pattern is common in training data. But for a less common expression like `1+1*2`, it may give you `4` because it has seen similar-looking patterns that returned that answer.

#### Lack of memory

LLM is stateless — it takes input and produces output, with no memory of previous interactions. Although modern chatbots appear to "remember" conversations, they are actually just feeding the conversation history back in as input each time.

#### Lack of tool use ability

LLM cannot interact with the outside world. It's a brain in a box. Most of the time, a chatbot can only give you text that you then have to manually copy into a code editor or terminal to execute.

#### Lack of self-reflection and self-correction ability

LLM cannot reflect on its own output and correct mistakes autonomously. Users have to sit there guiding it at every step — like a student who doesn't know if their answer is right or wrong without constantly asking the teacher.

---

With all the limitations above, LLM alone cannot handle many real-world scenarios. This is where agents come in.

## Why Agent (Claude Code) works so well

An agent is a system that combines LLM with other components — tools, memory, planning, self-reflection — to overcome those limitations. Let's look at how each component works.

![Agent Architecture](/images/post-14/agent.png "Agent Architecture")
<p style="color: gray; font-size: 1rem; text-align: center;">Agent Architecture</p>

### Planning

An agent can break down a complex task into smaller sub-tasks and solve them step by step. For example, if you ask an agent to book a flight, it can first ask for your destination and date, then search for flights, compare prices, and finally book.

The most fundamental planning technique is decomposing a complex goal into sub-tasks. This is achieved through prompting and architectural techniques such as:

- **Chain of Thought (CoT)** — prompting LLM to think step by step.
- **Tree of Thought (ToT)** — prompting LLM to generate multiple solution branches at each step and evaluate them to choose the best path.

### Memory

Memory is a must-have component for a useful agent. It's typically divided into two types.

#### Short-Term Memory

Short-term memory stores information from the current interaction by feeding previous messages back as input to the LLM. This lets the agent maintain conversational context and give more relevant responses.

Its limitation is the **context window** — LLM can only take a limited amount of input at once, so very long conversations may lose earlier context (techniques like context compression can help, but the limitation remains). Short-term memory is generally considered temporary and scoped to the current session.

#### Long-Term Memory

Long-term memory stores information that needs to persist across multiple interactions. There are two common approaches:

1. **Vector Database & RAG (Retrieval Augmented Generation)** — converts data into numerical vectors representing semantic meaning, stores them in a vector database, then retrieves the most relevant data at query time and feeds it to LLM as additional context.

2. **Local File System** — stores information in structured files (e.g., Markdown), then retrieves it when needed. For example, some popular bots store conversation history with dates as an index, and the LLM is prompted to retrieve relevant entries based on the user's input.

### Tool Use

LLM is a brain in a box — but an agent can use tools to interact with the outside world. For example, an agent asked to book a flight can call a flight API, get results, and act on them. At its core, tool use is just **prompt + API call**: the LLM is prompted to output a structured command when it needs a tool, and the agent parses that output and executes the corresponding API call, then feeds the result back to the LLM.

#### State-of-the-Art Tool Use

**1. Structured I/O**

While the fundamental mechanism is still prompt + API call, it has become much more reliable. Available tools are pre-defined and injected into the LLM's context as JSON/YAML schemas, and the LLM is prompted to respond in a strict structured format for any tool invocation. The latest models from OpenAI and Anthropic are also natively trained on millions of "tool-use trajectories," making them highly reliable at this.

**2. MCP (Model Context Protocol)**

Previously, every time you built a tool, you had to rewrite the "glue code" for every different agent framework (LangChain, AutoGen, etc.). MCP provides a universal interface — if a service (like Google Drive or a SQL database) has an MCP server, any MCP-compliant agent can plug into it instantly without custom code.

An MCP server typically contains three parts:
- **Tools (Executable)** — functions that the AI can call
- **Resources (Readable)** — static resources the AI can read (log files, database schemas, etc.)
- **Prompts (Templates)** — pre-defined prompts to help the AI use tools correctly

**3. ReAct Loop (Reason + Act)**

The ReAct loop is the industry-standard operating cycle for an autonomous agent. It consists of three steps that repeat until the task is complete:

- **Thought (Internal Reasoning)** — the agent analyzes the goal, reviews available information, and decides what it needs next.
- **Action (Tool Use)** — the agent selects a tool and provides the necessary parameters to run it.
- **Observation (External Feedback)** — the system executes the tool and feeds the result back into the LLM's context window.

This loop allows the agent to iteratively reason, act, and learn from the environment — making it effective at solving complex, multi-step problems.

## How to make it even smarter

AI is not a magic wand. It's a tool that can help you solve problems, but you have to know how to use it effectively. New models and agentic tools are emerging every day, but I believe the fundamental principles of using AI well remain the same. Here's what I've learned from my own wins and failures.

### Basic guidelines

**A clear goal is always helpful.**
An agent is a self-iterating system with a built-in ReAct loop — it only stops when it thinks it has completed the task. A clear goal helps the agent decide when to stop, making it more accurate and cost-effective.

**Plan before action.**
This is a common principle even for humans. Next time you ask an agent to do something, ask it to make a plan first. Review the plan yourself, then only ask it to execute once you're satisfied.

**Don't ask AI to build a house — ask it to build one brick at a time.**
Treat AI like an intern: you don't want to stifle its creativity, but you also don't want to give it so much freedom that it goes off track. Always think about whether a task is too large, and break it down if needed.

**Examples are always helpful.**
LLM is good at learning from (and copying) examples. If you have a clear idea of what the output should look like, give it an example — a screenshot for a UI task, or a code snippet for a function. The output will be much better than a textual description alone.

**Ask AI to take notes for you.**
Keep a changelog and ask the AI to update it every time a change is made. This way you can always review what's been done. You can also ask the AI to check its notes before taking any action.

**Never trust AI 100%.**
This sounds like a cliché but it's hard to follow in practice. When you start throwing tasks at AI, you'll be amazed by its power and gradually start trusting it more — sometimes blindly. It always feels good to keep clicking "accept" and move on to the next task. But remember: it's still a black-box, and when it makes mistakes, they can be serious ones.
