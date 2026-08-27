# My AI Development Workflow

Over time, my AI development workflow has become less about using a single AI coding tool and more about building a system around multiple agents. The main goal is simple: I want to be able to start several tasks, immediately understand what each agent is doing, get notified when something needs my attention, and continue working even when I'm away from my main computer.

My current setup revolves around **Herdr, OpenCode, a few custom skills and workflows, remote access through Termux, and an automated deployment environment**.

## Herdr as the Control Center

One of the main tools in my workflow is **Herdr**, a terminal multiplexer with strong integrations for AI CLI tools such as Codex, OpenCode, and others.

What makes it particularly useful for agent-based development is that it understands much more about the processes running inside the terminal than a traditional terminal multiplexer would.

For example, I can quickly see whether an agent instance is:

* still working;
* waiting for an answer;
* finished with its task; or
* sitting in some other state that needs my attention.

It also remembers processes and sessions that I've opened or closed, which makes managing several parallel agent sessions much easier.

Another surprisingly useful feature is sound notifications. When something finishes, I don't necessarily have to keep checking every terminal window. I can continue doing something else and get an audio signal when an agent reaches a point where I should look at it again.

Herdr also has good workflow support, which fits nicely with the way I use several AI agents at the same time.

Instead of thinking of the terminal as a place where I manually run commands, it starts becoming more like a dashboard for a collection of workers.

## OpenCode as My Main Coding Agent

My primary AI agent is **OpenCode**.

I've customized the environment with several skills so that the agent can do more than just modify code.

One example is a **Jira task creation skill**. Instead of opening Jira, creating an issue, filling out all the fields, and copying information around, I can simply describe the task to OpenCode.

It prepares the task for me, and once I'm happy with what it generated, it can add it to the board through my integration.

This is a small automation, but these are exactly the types of interruptions that add up during the day. If an agent can take care of the mechanical part while I focus on describing what needs to happen, the workflow becomes much faster.

## Isolating Agent Tasks with Git Worktrees

When I want an agent task to be completely isolated, I can invoke a custom **`/worktree` workflow**.

OpenCode asks Herdr to create a dedicated Git worktree and branch for that task, and then performs all edits, builds, and tests inside that isolated environment.

This makes parallel development much safer.

Multiple agents can work on the same repository at the same time without changing branches underneath each other, touching the same working directory, or interfering with uncommitted work in my main checkout.

That becomes particularly important once I'm running several agents in parallel. Without isolation, it's very easy for one task to change the state of the repository while another agent is still working with assumptions based on the previous state.

The workflow also includes some guardrails around Git operations.

Worktrees are never forcefully replaced or removed, and destructive Git operations are avoided. If a task spans multiple repositories, each repository gets its own isolated worktree rather than trying to mix everything into one shared environment.

So instead of every agent working in the same checkout, the structure becomes more like:

**Task → dedicated branch → dedicated worktree → edits → build → tests → review**

That gives each agent a clearly defined workspace and makes parallel AI development much more predictable.

## Notifications Instead of Watching Agents

One of the biggest problems with running AI coding agents is knowing when to come back to them.

An agent might work for several minutes and then suddenly:

* finish the task;
* encounter an error;
* ask a question;
* require permission; or
* wait for additional input.

Constantly switching back to check whether something happened defeats a lot of the benefit of running tasks asynchronously.

So I've connected OpenCode's completion and status events to my notification setup.

I'm using **ntfy.sh** to send notifications to both my PC and my phone. If an agent stops, finishes, or asks me something, I can get notified without actively watching the terminal.

That changes the workflow quite a bit.

Instead of:

> Start agent → stare at agent → answer → stare at agent again

it becomes:

> Start agent → continue working → get notified → respond when needed

That makes running several agents in parallel much more practical.

## A Separate Machine for Background Agent Work

I also have a separate PC/server running essentially the same OpenCode configuration.

This machine is useful for tasks that I want to completely separate from whatever I'm doing locally.

I can give the server one specific task and leave it working without the process interfering with my main development environment.

Because it uses the same notification setup, I don't need to actively monitor the server either. If the remote OpenCode instance finishes something or needs input, I still receive the notification on my phone.

This effectively gives me another independent worker.

My main machine can stay focused on the thing I'm actively developing, while the server handles another task in parallel.

## Managing Agents From My Phone

The remote setup becomes especially useful when combined with **SSH and Termux** on my phone.

Herdr works surprisingly well in this environment.

From my phone I can open Termux, SSH into the server, reconnect to the existing terminal environment, and move between the different tabs or agent sessions.

That means I don't need to be sitting at my desk just to answer a small question from an agent.

If I receive a notification while I'm away, I can:

1. open Termux;
2. SSH into my server;
3. jump into the relevant Herdr session;
4. check what OpenCode is asking;
5. give it another prompt; and
6. let it continue working.

I'm obviously not going to do serious development from a phone keyboard, but for steering agents it works surprisingly well.

Most of the time I don't need to write code myself. I just need to inspect what happened, answer a question, or give the agent its next instruction.

## Automatically Deploying Agent Changes

Once an agent has changed something, I still need a good way to verify that the result actually works.

For that I have another part of the workflow: a deployment script.

When I like the changes, I can run the script and have it pull the relevant version, build the necessary Docker containers, and publish the result to my server.

This gives me a real environment where I can test what the agent created.

Instead of only looking at a diff and assuming that everything works, I can actually open the application and verify:

* whether the feature behaves correctly;
* whether the UI looks right;
* whether anything broke;
* whether the containers build properly; and
* whether the result matches what I originally asked for.

Because the deployment is accessible through my own domain, I can also send the result to someone else and let them test or review it.

That makes the feedback loop much shorter.

The workflow becomes:

**Prompt → isolated implementation → notification → review → deployment → real-world testing → next prompt**

## Giving Agents Structural Code Context

Another part of the workflow I'm interested in is giving agents more than just raw file contents.

An AI coding agent can usually search a repository and inspect files on its own, but there is a difference between *finding code* and actually understanding the structure of a codebase.

The more structural context I can give an agent — how modules relate to each other, where important boundaries are, which components depend on which others, and where a change is likely to have consequences — the less time it has to spend rediscovering that information for every task.

This becomes increasingly useful as the codebase grows and as more agents work on it in parallel.

The goal is not necessarily to dump more context into every prompt. It's to make the right context easier for an agent to discover when it needs it.

## The Bigger Idea: Managing Agents Instead of Watching Them

The most important part of this setup isn't any individual tool.

It's the way the tools work together.

Herdr gives me visibility into all of the running agent sessions. OpenCode does most of the actual coding work. Custom skills connect the agent to things like Jira. The `/worktree` workflow gives individual tasks isolated development environments. **ntfy.sh** tells me when human input is required. My second machine gives me another independent environment for long-running tasks. Termux lets me steer everything remotely. And my deployment scripts turn agent output into something I can immediately test.

The result is that I spend less time watching AI work and more time **managing what should happen next**.

That's the direction I find most interesting with AI-assisted development.

The real productivity gain doesn't come from asking an AI to write a function slightly faster. It comes from building an environment where multiple agents can work independently, operate safely in parallel, report back when necessary, and move their work all the way from an idea to something I can actually test.

At that point, the terminal stops feeling like a place where I'm typing commands one by one.

It starts feeling more like an operations console for a small team of AI workers.
