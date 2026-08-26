---
title: "How I use LLMs"
author: "Sean Liu"
date: 2026-08-23T00:29:00-05:00
categories: ["cs", "programming"]
draft: false
---

> Human statement: I do not use LLMs in any of my blog posts for writing nor editing (though obviously it is the subject matter).

## Introduction

Adding my entry to the pool of posts on [Hacker News](https://news.ycombinator.com/) and [lobste.rs](lobste.rs) around AI usage in the context of software engineering. This post is about how I currently think about the use of LLMs to write code and help software engineers' lives easier, including common patterns I've seen work in practice and pitfalls I've observed many (yours truly included) fall into when trying to get agents to work. I hope at least some of the ideas here are new, and if not hopefully I shed them in a newer light; failing _that_, that they are at least organised in a better way.

My stance on this topic is not too controversial I hope: I think that they represent a significant change in how we work and that they can be a tool to significantly boost developer productivity. They're good enough to do rote wiring tasks, bounce ideas off of, and make throaway visualisations. On the other hand, I do not think them good enough to make fine-grained decisions or design large systems built for any sort of timeframe. They're like steam locomotives moreso than bicycles: _if_ you set the tracks right, _if_ you clear all obstacles, and _if_ you know exactly where you want to go, then you'll get to your destination faster than if you rode a bike. Indeed, there are a plethora of usecases for trains over bicycles, but bicycles yield more control to the operator, are more nimble, require more skill to manouevre, and can get you to more places. I think it is wrong to delegate everything to LLMs (though it is certainly a goal to strive towards) as these agents are hardly a replacement for proper human thought; conversely it is Luddite and counter-productive to refuse to use LLMs in writing code in a professional setting. This blog post aims to explore good patterns in setting these train tracks, without going into _if_ they should be set.

Finally, I'd like to state that most of everything I'm writing about here is just standard "good" engineering practice that is applied in this context. That having good tooling, visualisations, and a good data model are beneficial to the developer process is not a new concept; but I believe these are the easiest to get started with practically, and give outsized returns with comparatively low effort.

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

### Determinism and CLI tooling

The "bring Claude into our repo story" goes like this: 

0. You clone your repository. You ask Claude to do something. It has no context. It falls on its face and you're unhappy with it. 
1. You realise that you need good documentation to serve as context. You (or Claude) write docs. Claude is a little better but now its context is often bloated; common things are pretty unpredictable.
2. You then identify common things that you'd like Claude to do repetitively. For example, running CI, reviewing PRs, or gathering context. You package these as custom [skills](https://support.claude.com/en/articles/12512176-what-are-skills) to invoke. This makes sure your prompts are deterministic, but you soon find that your context again bloats, and the commands you ask Claude to run... don't always get ran correctly. 
3. You then think to gather all the deterministic parts into a bash script and ask Claude to call these scripts in your skills. This effectively makes your skills deterministic _and_ significantly reduces context bloat.

Sound familiar? This is nothing new, not even with agents. This is **exactly how devs work**! Devs will onboard onto a new project, get acquainted, subsequently frustrated with some rote work, and build tools to help them automate. This is exactly the same. So why not make step 3 the standard, so that humans and agents can both benefit? I propose that the **CLI** itself should be a first-class citizen instead of a ragtag bunch of bash/python scripts referenced to by a skill. If you are primarily using agents to interact with the code, it is worth (a) defining the contact surface by defining all possible operations in the CLI and then asking your LLM to _only_ run commands in the CLI, and (b) making sure your CLI is easy to work with. Oversimplifying of course but this is hopefully a simple _modus ponens_: if the only sanctioned way for agents to interact with things in your repo is through the CLI, and your CLI is implemented well, then your agents will be predictable and correct. Moreover, it won't have to spend valuable time and tokens figuring out _what_'s correct to run, or run into any of the multitude of failure modes possible (e.g., your dataset's not partitioned in the right way so queries subtly take much longer, assuming that a command appends instead of upserts so your data gets accidentally deleted, etc.) because it just has to run a single CLI command. 

For example, my CLI in my phylogenetics research repo currently looks something like

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

The point is that Claude is _very_ good at doing the CLI wiring! It's up to the human to define which operations you'd like to expose, the parameterisations to allow, etc., but after that Claude is very good at implementing the CLI spec. This is an example of using Claude to build scaffolding to make its operations more reliable - this Python `Typer/Rich` scaffolding is low-risk boilerplate code that Claude can few-shot without much issue, but it boosts the stability of Claude in the repo considerably. Such cheap and tools are now viable because [Code is Cheap](https://nadh.in/blog/code-is-cheap/); as nobody cares about this wiring it's fine it's AI-generated slop. A few notes: 

- This is runnable with `uv run` so Claude does not have to worry about requirements, virtual environments, etc. For Python CLIs I recommend using [uv](https://docs.astral.sh/uv/); you could also make your CLIs blazing fast and use a compiled language such as C++ or Rust.
- The commands section basically list all the operations that I'd like to be able to do: `simulation`, `infer`, `score`, etc. Underneath they fan out to the aforementioned handwritten bash scripts - but Claude doesn't need to know that, nor where they are situated, nor what order to run them in. 
- paired with a short skill explaining what these commands are, how to use them, etc. you get something that is very powerful - Claude is smart enough to reason about what you want from natural language, and then just run the necessary commands.

### Observability and the data model

Another thing I improved with this setup is to have a very explicit _data model_ for everything. The data model is how the human intends for the system to be reasoned about. It is a statement of what the problem space is and what we care about. What's more, having an explicit data model lets Claude very easily have all the context it needs without having to go search everything up in different locations up. Again this contributes to everything being fast, context-efficient, and deterministic. Nothing groundbreaking here, just that the calculus has changed such that having a good data model is both easy to do _and_ gives immense benefits.

In my particular case, I applied this in two ways: a YAML-based input model to specify an entire experiment, and making sure my outputs were stored in a predictable place + ensuring everything was trivially joinable. This YAML-based input model might look something like:

```yaml
default_sim_configs: &default_sim_configs
  poly: high
  homoplasy_factor: 0.1
  n_chars: 80

experiment_folder: experiments/smoke_test
simulation:
  n_taxa: 30
  n_trees: 1
  simulation_params:
    # condition 1: high polymorphism
    - <<: *default_sim_configs
      poly: high
    # condition 2: low polymorphism
    - <<: *default_sim_configs
      poly: low

methods:
  mp4: {}
  astral_3:
    is_exact: False
    bipartition_strategies:
      - mp4_trees
      - ga_trees
  wastral: {} 
```

The point of this file being that it is meant to be the singular source of truth to configure the CLI. This makes it very easy for Claude (or a human) to tell what is going on in a particular experiment instead of scouring a bunch logs / relying on file structure.

With the output, it's just standard good data modelling: have a set of hopefully universal primary dimensions to join on, have informative facts, link to raw files, etc. I'm not here to tell you how to do data engineering properly (I'd probably be the last one you'd ask), I'm saying that even for personal projects, if you're enlisting the help of LLMs it's definitely worth it to try and establish a good data model. The only reason one wouldn't do it for a personal project before was because it would be too much grunt work -- but hey guess what LLMs are good at? With a short document to explain the semantics of columns and join keys, Claude becomes _very_ good at understanding the problem space.

### Another example: `seba` 

Another example I'd like to give is [`seba`](https://github.com/xSeanliux/seba), a tutor skill I'm working on, and that I'm currently interacting with to go through the material in the fantastic [Category Theory Illustrated](https://abuseofnotation.github.io/category-theory-illustrated/). It follows a very similar pattern of a deterministic CLI + a skill to prime agents to interact with the CLI. It tries to solve the problem of "long-term learning," i.e., if you want to learn a whole _syllabus_ through Claude, you're most likely going to need more than one session (or equivalently clear the same session). How do you persist learning progress through sessions? How does Claude know what to go over in each session? My solution is thus: 

- When you first start a new subject, you give it some subject matter. Claude generates a syllabus of "goals" in YAML format (data model!) that might look something like 

```yaml
goal: Understand introductory probability     
subject: probability                          
concepts:
  - id: sample-spaces                          
    name: Sample spaces and events             
    prereqs: []                                
    soft_prereqs: []                           
    confusable_with: []                        
    kc_type: concept                           
    sources: []                                
    status: unseen                             
    est_sessions: 1                            
  - id: conditional-probability
    name: Conditional probability and Bayes
    prereqs: [sample-spaces]                   
    soft_prereqs: []
    confusable_with: []
    kc_type: concept
    sources: []
    status: unseen
    est_sessions: 2
```

- Every session is focused on a new goal, and Claude is instructed to run `seba status` to see statuses of all goals, then to pick one and run `seba start GOAL`. During a session Claude may mark a goal with its difficulty (paired with a spaced repetition module to maximise recall) / add notes such as what went well, common confusion points, etc. to a Markdown file. All of these operations have a dedicated CLI command, so that Claude only has to know to use these commands; everything else is already structured.

## Throwaway code as dev tools 

The argument I gave above on why CLIs are conducive to LLM development and usage (low-risk code that is OK to be slop and which supports the main, hopefully not slop, function of the codebase) can be extended to other parts of the developer experience. There is a wide gap between "just let LLMs generate code and I'll merge it in" and "I'm handwriting everything." LLMs offer _optionality_: the option to not care about the code. It does not mean you care about no code (this is called _irresponsibility_). Indeed, I think to be effective, an engineer needs to very consciously make the tradeoff between code that she cares about, and code she does not. 

I do not think it controversial to say that the understanding of code, the intent behind development, the decision making process, and human thought behind codebases should not be outsourced to machines. However, much of that can be made more efficient if you allow some LLM intervention. For example, generating dashboards for designs to share with teammates, or for observability around your systems are all very good reasons for "throwaway code," you don't need to care about the HTML frontend nor how it's hosting the HTTPS website, just that you have a usable interface (and perhaps that it's pulling from the right APIs). Once again one sees productivity increases for next to no effort - Claude is more than good enough at doing rote wiring work and making a pretty(ish) frontend to display data.

Reviewing code is another front of this - suppose that you're working on something that you _do_ want to fully control and understand. I think it's acceptable for such changes to be almost entirely AI-generated as long as you are able to explain all of it and have read the entire thing. It is not fun to read a PR diff the way GitHub presents it - in sorted file order. Instead, it is much better if it is presented in some other way (e.g., call stack order, interactive walkthroughs, etc). This is an example of using an LLM to boost understanding - it presents its code in a more human-friendly way so that it can be reviewed and picked apart. Skills like `/code-review`, [`/ponytail`](https://github.com/DietrichGebert/ponytail), &co can only take you so far, because they are ultimately _local optimisers_, making code look good locally. It does not know _why_ you designed the architecture this way (of course you can write it in the spec but does anybody actually do that? Does anybody write down _every_ minute decision that affects the architecture?). In short, it does not understand _intent_.

## Code as intent 

When you write a piece of software, you have all sorts of things in mind. How your code should behave and that it should pass unit tests / integration tests / linters is only the first layer. I subscribe to Naur's view that [programming is theory building](https://pages.cs.wisc.edu/~remzi/Naur.pdf), and encoded within the codebase are innumerable tiny decisions on how to organise code, subtle tradeoffs in performance, readability, extensibility, etc., that are just not discernable by unit tests alone. Tests and asserts you don't write matter as much those that you _do_ (because they might represent some underlying assumption you have that you know is valid, for example), repeated dataclasses might not be bloat but a conscious statement on module boundaries, and architecture designs often make it easier or more difficult for a piece of software to be extended in various ways, implicitly encoding the engineer's vision. None of this can be captured by benchmarks or a skill because (a) this is often a matter of taste and hence subjective, and more fundamentally, (b) you cannot test for what is expected to be _absent_. A simple function with 1000 unit tests and way too many guards in its code will work but will not sufficiently express the _intent_ of the function (not to mention its performance).

One might make the counterargument that this is only valid insofar as humans need to read the code, and models get more powerful, one will not need to read the code just like we do not (often) read compiled machine code. The analogy here is that the C++/machine code of yesteryear is directly comparable to the Markdown/actual C++ of the present. Sure - you can try to describe everything comprehensively in a spec in natural language, but one often finds that this spec is not so far removed from the code itself. That is not to say you should not do this sort of spec-driven development, just that you have to take some care in doing so. I find it most useful to jot down an existing design / write pseudocode -- and then iterate with Claude to iron things out and then turn it into a spec to implement, trusting in its ability to do the annoying wiring up. Seeding Claude with a design you already know drastically improves your ability to read and critique its code later on, though it often will still make mistakes. Being able to identify these subtle mistakes and critique an architecture is why I still think a proper (not necessarily formal) CS education is still (and even more) valuable: now that writing the code itself is easy the onus on all devs is in the decision process.

These sorts of mistakes are often not due to "the model being dumb" (though it does happen), but that it's trying to make a tradeoff based on limited/guessed/wrong information. Not saying that you can't anticipate every assumption it'll make and bake it into the spec, just saying that at some point you run into diminishing returns, and if you successfully do that you've written, for all intents and purposes, code anyway.

## Tending the garden

I did not come up with this analogy myself, but thought it'd be useful to share.

The first analogy is tending a garden: traditional (i.e., handwritten) software is very deliberate and slow, it'll get the job done and you'll come out on the other side after a lot of toil, and skilled landscapers will end up with a beautiful garden. Now imagine you want to expand to managing a much larger garden, and you come into posession of a large quantity of extremely potent bonemeal. These stochastic code parrots are exactly like that bonemeal: it is very powerful but hard to control and can easily make your life a lot worse if not done correctly. The misuse of this bonemeal will turn your garden into an anarchic wastefield, but if you know to construct frames and supports around which you guide your plants to grow, then you have a very effective way to manage a bigger estate, able to complete larger projects in a fraction of the time as before. With traditional gardening, you did not have to construct any frames or scaffolds but to be able to use bonemeal you have to invest in extra steps to keep your plants behaving; in other words, you changed the way you work to make the best use of a much more powerful tool.

**TL;DR** my opinion is that AI-assisted engineering is here to stay, and that it offers us optionality to decide which parts of code we care about vs it just being scaffolding / throwaway dashboards. Having a clean set of (CLI) tools and a transparent data model so that LLMs and humans can more easily interact with the code base pays dividends; with how good AIs are at wiring things up these are almost free wins that even codebases with a single contributor can benefit from. Lastly, I do not think that models are good enough to manage larger systems that people care about just yet for structural reasons - *own your design and your thought process*! 