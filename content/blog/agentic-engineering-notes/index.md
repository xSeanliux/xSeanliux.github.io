---
title: "How I use LLMs"
author: "Sean Liu"
date: 2026-08-23T00:29:00-05:00
categories: ["cs", "programming"]
draft: false
---

## Introduction

I've been seeing a bunch of posts on [Hacker News](https://news.ycombinator.com/) and [lobste.rs](lobste.rs), so thought it might be worth giving my 2¢. This post is about how I've gradually come to think about the use of LLMs to write code and help software engineers' (read: my) life easier, common patterns I've seen people adopt, and pitfalls I've observed many -- yours truly included -- fall into when trying to use something like [Claude Code](https://claude.ai) to aide in software development / vibecoding. I will use two recent personal projects as an example, but of course one should be able to apply this in a more professional setting. Like you no doubt, I have also read quite a few posts on this topic; I will also quote some posts I (dis)agree with in this post. I hope at least some of the ideas here are new, and if not hopefully I shed them in a newer light; failing _that_, that they are at least organised in a better way.

My stance on the use of LLMs in engineering is not too controversial I hope: I think that they represent a significant change in how we work, and that they can be a tool to significantly boost developer productivity. I do not think that these machines are going to replace humans any time soon. I think of them as trains moreso than bicycles: _if_ you set the tracks right, _if_ you clear all obstacles, _if_ you know exactly where you want to go, then you'll get to your destination faster than if you were on a bicycle. Indeed, there are a plethora of usecases for trains over bicycles, but bicycles yield more control to the operator, are more nimble, require more skill to manouevre, and can get you to more places. I think it is wrong to want to delegate all work to LLMs (though it is certainly a goal to strive towards) as matrix multiplication is hardly a replacement for human thought; conversely it is Luddite and counter-productive to refuse to use LLMs in writing code in a professional setting (one is of course free to do whatever they want in private). This blog post is about _how_ I am setting these train tracks, not if they should be set.

## Bones and meat & "AI native engineering" 

<small> god I hate buzzwords </small>

As part of my research in linguistic phylogenetics, I had a repo full of ad hoc scripts to run tree & population simulations, inference code to try out different inference methods on these simulated data, SLURM wrappers to submit these scripts, scoring scripts to score output, metrics, all sort of things. It was not obvious how things worked: it had some unit testing in Python but coverage was atrocious, Python type annotations were spotty, repo structure and input/output placement/formats were more conventions than anything else. Furthermore, there were hidden dependencies in my scripts (as in, if you ran certain scripts in the wrong order you'd just get silently wrong output). 

This was obviously not production code - it was the way it was because 

- I was a college student and I had classes to take, social functions to attend, etc. and research was one of many responsibilities. 
- All of this code was handwritten, except save for a couple of Bash scripts for which I used ChatGPT to generate some argument parsing boilerplate.
- This research codebase was more built for "can you do XYZ experiment," and not anybody else to contribute to my codebase. Thus as long as I knew where everything was and how things were supposed to be ran (and a dozen other things), experiments ran just fine. 

All in all, quite bad! After graduating in 2025, I started working full-time and had even less time to devote to this research. I already had a re-design of the repo in-the-works, and was slowly sketching things out and writing code, when in July 2026 (almost a year later!) I decided that this was just too slow and decided to try out Claude Code on this.

The premise was simple but ultimately took me more than a month to get done, even with Fable 5. I did not simply want to point the repo at Claude and 

```
/ultracode hey Claude, make this repo better. Make no mistakes.
```

Instead, I took the time to design code, make sure it expresses the correct data model, has tools, etc. The end goal of this being that Claude should be able to run things inside the repo itself without much outside help, and I can be sure of its correctness. Obviously this is more than a skill or some repo docs (you and I both know those are AI generated and nobody knows how accurate they are). 

This brings me to my first point, determinism and CLI tooling.

## Determinism and CLI tooling

The "bring Claude into our repo story" goes like this: 

0. You clone your repository. You ask Claude to do something. It has no context. It falls on its face and you're unhappy with it. 
1. You realise that you need good documentation to serve as context. You (or Claude) write docs. Claude is a little better but now its context is often bloated; common things are pretty unpredictable.
2. You then identify common things that you'd like Claude to do repetitively. For example, running CI, reviewing PRs, or gathering context. You package these as custom [skills](https://support.claude.com/en/articles/12512176-what-are-skills) to invoke. This makes sure your prompts are deterministic, but you soon find that your context again bloats, and the commands you ask Claude to run... don't always get ran correctly. 
3. You then think to gather all the deterministic parts into a bash script and ask Claude to call these scripts in your skills. This effectively makes your skills deterministic _and_ significantly reduces context bloat.

Sound familiar? Well, this is nothing new, not even with agents. This is **exactly how devs do things**! Devs will onboard onto a new project, slowly get acquainted, get frustrated, and build tools to help them with their work. This is exactly the same. So why not make step 3 the standard, so that humans and agents can both benefit? Indeed, I propose that the **CLI** itself should be a first-class citizen instead of a ragtag bunch of bash/python scripts. If you are primarily using agents to interact with the code, it is worth a) defining the contact surface by defining all possible operations in the CLI and then asking your LLM to _only_ run commands in the CLI, and b) making sure your CLI is good. Oversimplifying of course but this is hopefully a simple _modus ponens_: if the only sanctioned way for agents to interact with things in your repo are read commands & your CLI, and your CLI contains sanctioned operations that are implemented correctly, then your agents will be predictable and correct. Moreover, it won't have to spend valuable time and tokens figuring out _what_'s correct to run, or run into any of the multitude of failure modes possible (e.g., your dataset's not partitioned in the right way so queries subtly take much longer, assuming that a command appends instead of upserts so your data gets accidentally deleted, etc.) because it just has to run a single CLI command. 

For example, my CLI currently looks something like

```
(base) *[main]$ uv run -m  scripts.py.cli.main --help
                                                                                                                                
 Usage: python -m scripts.py.cli.main [OPTIONS] COMMAND [ARGS]...                                                               
                                                                                                                                
╭─ Options ────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ --install-completion          Install completion for the current shell.                                                      │
│ --show-completion             Show completion for the current shell, to copy it or customize the installation.               │
│ --help                        Show this message and exit.                                                                    │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╯
╭─ Commands ───────────────────────────────────────────────────────────────────────────────────────────────────────────────────╮
│ simulation                                                                                                                   │
│ infer       Atomic inference on one dataset; renders the InferenceResult.                                                    │
│ score       RF-score one estimate against a reference Newick.                                                                │
│ summarize   Consensus-summarize a tree set to a single Newick.                                                               │
│ experiment                                                                                                                   │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────╯
```

This is not perfect, but notice that 

- This is runnable with `uv run` so Claude does not have to worry about requirements, virtual environments, etc. For Python CLIs I recommend using [uv](https://docs.astral.sh/uv/); you could also make your CLIs blazing fast and use a compiled language such as C++ or Rust.
- The commands section basically list all the operations that I'd like to be able to do: `simulation`, `infer`, `score`, etc. Underneath they fan out to the aforementioned handwritten bash scripts - but Claude doesn't need to know that, nor where they are situated, nor what order to run them in. 
- paired with a short skill explaining what these commands are, how to use them, etc. you get something that is very powerful - Claude is smart enough to reason about what you want from natural language, and then just run the necessary commands.

### Observability and the data model

Another thing I improved with this setup is to have a very explicit _data model_ for everything. 

- yaml based input model 
- output csv joinable

## Code as intent 

1. Agents do not know intent 
2. There are tons of way to pass tests and have the same behaviour, but how it is written in the code expresses intent around how the code is intended to grow, how it is intended to _not_ grow, assumptions around modes of failure are expected (and conversely about what kinds of failures are _not_ expected).
