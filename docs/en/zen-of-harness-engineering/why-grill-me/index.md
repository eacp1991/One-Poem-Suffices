![zen-of-harness-engineering](./assets/zen-of-harness-engineering.png)

# How to Think About the `grill-me` Skill?

A few days ago, I answered a question on Zhihu about my experience using the `grill-me` Skill.

[`/grill-me`](https://github.com/mattpocock/skills/blob/main/skills/productivity/grill-me/SKILL.md) is an open-source AI Skill created by Matt Pocock. Its core logic is simple: before the Agent starts implementing anything, it conducts a round of in-depth questioning about the requirements. One by one, the Agent asks what you actually want, what the edge cases are, and which parts haven't been thought through yet. Each question usually comes with a recommended answer; once you confirm or adjust it, the Agent moves on to the next one.

I use this tool frequently and have recommended it to many people.

Before I had this Skill, I usually appended a line like this to the end of an instruction:

!!! quote "A line I used to append to the end of my Instructions"

    **If anything in my requirements or design is unclear, or if there is anything you want to push back on, please ask me directly.**

Typing the same instruction repeatedly was tedious, so I eventually wrote a simple prompt for it. By comparison, the instructions in `grill-me` are very lightweight, much more convenient to use, and work surprisingly well.

On the surface, **it solves the problem of people getting too impatient in Agentic Coding.** Building a demo is so fast now that many people skip necessary system design and just tell the Agent to start writing code first. `grill-me` slows the process down slightly, forcing you to do some necessary design work before implementation begins.

However, I prefer to view it as a unified **Harness requirement**. Regardless of whether a task involves coding, as long as the execution path is sufficiently long and the outcome could go in multiple directions, humans need to inject their preferences and key judgments before the Agent takes large-scale action.

## How Is It Different from Regular Brainstorming?

Compared to the Brainstorm skill in Superpowers, `grill-me` asks much more detailed questions and is better suited for exploring a specific branch in a DFS-like manner. If a question is worth unpacking on its own, I sometimes use `/branch` to enter a new session, write the conclusions of the discussion back to the filesystem, and then switch back to the previous session to continue along the main thread.

!!! warning "Some Questions Can Simply Be Skipped"

    It also asks unreasonable questions from time to time. For example, some outcomes are actually outside your control, or simply cannot be determined at the current stage. In those cases, you can just skip them. Conversely, these unanswerable questions also serve as a reminder that there is a gap between your expected level of completeness and what you can actually control.

A good questioning Skill shouldn't dump every question onto the human, either. If the answer already exists in the codebase or documentation, the Agent should read it on its own. Beyond asking questions, [`/grill-with-docs`](https://github.com/mattpocock/skills/blob/main/skills/engineering/grill-with-docs/SKILL.md) also maintains the project's domain model, glossary, and ADRs. For projects maintained over a longer period, this approach is a better fit.

Because I maintain a Shared Context Folder myself, **the questions `grill-me` asks me** are usually no longer big-picture questions like tech stack selection, but rather small yet **critical decision points**.

!!! tip "Maintain a Shared Context Folder"

    I strongly recommend keeping a Context Folder when maintaining a repo, and requiring the Agent to record the Design and Implementation Plan before starting implementation.

Connecting this to Harness Engineering, I think about this type of Skill on three levels.

## 1. From a PCA Perspective: Injecting Taste

Going back to a point I made in [Claude Code Source Distillation: A Practical Record of Harness Engineering](../claudecode-distillation-practice/): **the most important thing in Agentic Coding is actually your personal Taste.**

Why? Because in the absence of specifications, an Agent's Action Space is much **wider** than what human intuition assumes. A crucial role for humans in Agentic Coding is to inject their own Taste into the decision-making process and **prune** the Action Space.

I think of Skills like `grill-me` as **the process of extracting basis vectors in PCA (Principal Component Analysis)** (this analogy is just an aid for understanding; no need to overextend it).

Let me reuse an image from another blog post to convey the general idea~

![PCA: Projecting from High-Dimensional Code Space to Low-Rank Pattern Space](../claudecode-distillation-practice/assets/fig3-pca-projection.png)

When you want to build something, the idea in your head is more like a high-dimensional **Latent Vector**: vague, not fully unpacked, and full of implicit assumptions. **In the past, implementation was much slower than it is today**, so you could gradually decode that latent vector during the implementation process itself. Now, however, implementation happens too fast. If you don't establish control right at the beginning, the result will quickly drift away from what you envisioned, and the cost of reverting will be high. Because you didn't write the code yourself, you might not even have enough information to tell where the drift began, leaving you no choice but to scrap the whole thing and start over.

Through continuous questioning, `grill-me` helps you identify the few most critical **basis vectors**, which are your core design decisions. Once those basis vectors are pinned down, you perform a **compressed projection**, allowing a complex and uncertain problem space to gradually **converge into a structured, executable plan**.

In other words, ideas in the human mind are latent, and questioning makes the critical parts **explicit**. When an idea stays in your head, it often feels like "yeah, that's pretty much how it works." But once you unpack the problem in concrete terms, you realize there are actually several completely different ways to solve it. And **that is precisely where you need to make design decisions**.

## 2. From a Cybernetics Perspective: Reducing Uncertainty

From a cybernetics perspective, the value of `grill-me` lies in **reducing uncertainty during execution**.

When doing design work, people often operate on a vague intuition, feeling that "it should be right" or "the machine should understand what I mean." In reality, that is rarely the case. Questioning explicitly unpacks the critical issues, allowing you to actively choose and course-correct so that the subsequent execution path stays as aligned as possible.

**This is especially important for Long-horizon Tasks. The longer the execution path, the more likely a tiny directional deviation early on will be continuously amplified later. Necessary human Taste Injection and checkpoint control are immensely valuable to the final outcome. Otherwise, the Agent is simply burning through tokens faster along the wrong path.**

That's why it's worth investing significant time in Harness Engineering. These investments may seem to slow down the start of execution, but they noticeably improve the efficiency of the production process that follows.

## 3. My Personal Practice: A Three-Layer Control System

I currently optimize the Hand-off process in Agentic Coding primarily through a three-layer mechanism.

### 3.1 Shared Context Folder

In every GitHub project I maintain long-term, I keep a Shared Context Folder, which mainly contains:

- **Design Docs**: core architecture and design decisions
- **Preferences**: coding style and tech stack preferences
- **Progress**: records of key implementations and updates
- **Workflows as Skills**: standardized workflows are themselves Context, such as how to organize Context files or how to create an Issue

The goal here is not to hand the Agent a thousand-page manual, but to let it know where to look for long-term facts about the project. Before implementation, the Agent first updates the Design and Implementation Plan; after implementation is finished, it writes key progress and new decisions back into the folder. In this way, the filesystem becomes shared project memory across sessions.

For more on this, you can refer to the discussion on Skills and Context in [Agent Skills: One Poem Suffices](../../one-poem-suffices/agent-skills/).

### 3.2 Using GitHub Issues and PRs for Tracking and Hand-off

When I propose a new requirement, I have the Agent create an Issue based on our existing Skills. This Issue serves as the initial **Hand-off Prompt**, containing the requirement context, design decisions, execution boundaries, **verification workflow, and what needs to be written back upon completion**.

Issues and PRs also act as entry points for progressive disclosure. Later on, there is no need to copy a huge block of Context into every session; I just reference `#xxx`, and the Agent can follow the index to read the information relevant to the current task.

### 3.3 Finalizing with `grill-me`-Style Interaction

Before the Agent actually starts executing, I use `grill-me`-style interaction to finalize the design:

- Part of the conclusions is written directly back into the Issue Body to serve as the basis for subsequent tracking
- The other part is organized into a structured **Hand-off Prompt**, used for delegating tasks to a clean session or a sub-agent

This workflow mainly optimizes task hand-off and parallel efficiency. When a new Agent takes over, it doesn't need to know the entire prior chat history; it only needs to know its Role, its Task, and where the relevant Context lives in the filesystem. For a more complete discussion on when it makes sense to split work across multiple Agents, see [Thinking in Context: When Do You Need Multi-Agent Systems?](../../thinking-in-context/when-multi-agent/).

## Closing Thoughts

At this point, `grill-me` is to me more than just a prompt that "makes the Agent ask a few more questions." It is more like a control node in the Harness: before execution begins, it turns the latent Taste in the human mind into recordable, verifiable, and hand-off-ready design decisions.

The Shared Context Folder preserves long-term memory, Issues and PRs handle Tracking and Hand-off, and `grill-me`-style interaction eliminates critical ambiguities before action is taken. All three layers ultimately address the same problem: **as Agents act faster and faster, how can humans continue to stay accountable for the direction?**

These are just some of my personal practices. I hope to write a more complete piece later on: *Harness Engineering: One Poem Suffices*.

## Related Reading

- [Claude Code Source Distillation: A Practical Record of Harness Engineering](../claudecode-distillation-practice/)
- [Thinking in Context: When Do You Need Multi-Agent Systems?](../../thinking-in-context/when-multi-agent/)
- [Context Engineering: One Poem Suffices](../../../one-poem-suffices/context-engineering/)
- [Agent Skills: One Poem Suffices](../../one-poem-suffices/agent-skills/)
