---
title: "Using AI coding tools for vulnerability research"
date: 2026-03-12T00:00:00+01:00
---
Published: 2026-03-12
<!--more-->

> As usually, any information in this post will be obsolete with release of new "industry-disruptive" model (which happens every two weeks :)).


Over the past months, I have been building various AI SAST and DAST prototypes for vulnerability research and bug hunting. I tried different approaches, models, and tools, but spoiler alert: I was not able to build **anything better than the default Clade Code**.

It might be a skill issue, or it may be because *nobody* has created a better AI security tool than the default Claude with Opus 4.6.

This post discusses my findings from using general AI coding tools such as Claude, Gemini-cli, and Codex. It assumes the reader is familiar with these tools and how they work internally.

That said, I gathered some insights on how to use and optimize them to find vulnerabilities.

During my experiments, I found **zero-day vulnerabilities** in widely used open-source projects, which I will hopefully cover in future.

As a result of my experiments, I wholeheartedly believe that AI **could** replace at least 40% of the work of a human vulnerability researcher, mainly because it reads code more efficiently than humans do.

Validation may be the biggest portion of manual work of future security engineers, as LLMs will be replacing some parts of traditional vulnerability discovery like code review.

## Prompting

Although prompting might seem like pseudoscience, it is, unfortunately, the most important part of your setup. Even the best current models cannot construct a reasonable threat model. Even in well-known, well-documented codebases like GitLab or Mattermost, the models struggle to differentiate between suspicious-sounding features and actual vulnerabilities.

For a real-world example: if code has a function named `is_whitelisted()` that can be bypassed, the models will report it as finding even if allowlisting is a function on the frontend just for backward compatibility. Naming might be misleading, but it's not a vulnerability.

It is therefore crucial to carefully state the organization's or project's threat model and, ideally, provide a prompt such as: `Prioritize finding actual vulnerabilities over false positives.`

The default ratio of false positives to actual vulnerabilities is 1 to 50. The. The best thing is to set up a live instance of the software and instruct the model to always `validate and create PoC with simple human steps` for each finding.

Unfortunately, there will still be a 1-to-10 ratio of false positives to actual vulnerabilities.

[Recent papers](https://arxiv.org/pdf/2602.11988) suggest that using project-level files like `AGENTS.md` actually makes performance worse. Let the model do its own greping and reading, and it will learn on its own. Do not explain the source code, because you will probably be worse than the model itself.

Finally, bear in mind that LLMs in general perform better on small, well-defined tasks, so look for past CVEs to describe what the model should look for. Every company releases prompt engineering guidelines for specific models; make sure to review them.

For example:
- https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/overview
- https://developers.openai.com/cookbook/examples/gpt-5/gpt-5-2_prompting_guide/

## Determinism

There is [inherent randomness](https://thinkingmachines.ai/blog/defeating-nondeterminism-in-llm-inference/) in how models interpret prompts and generate responses. Even with careful prompting, the model may perform differently on the same inputs.

I advise letting models run multiple times for more consistent results. Always remember to clear context between runs, because context pollution will lead to regression over the same findings

For example here is architecture of tool I built for finding "interesting" scopes in code:

![determinism](/web/Screenshot%20From%202026-03-06%2010-34-26.png)

It just runs again and again multiple agents which vote if a scope is interesting (read vulnerable) or not.

Other very important aspect which you have to consider during run of agent in loop is memory management.

For example, if you are running agent more than once with same prompt and clean context, you have to be careful about what the model remembers from previous runs.

Always look out for memory files (such as `MEMORY.md`) in the repo, because, as statistical models, even if you say "Stack overflows are not considered security vulnerabilities", the model will still, at best, convolute messages about unnecessary comments on the findings and, in the worst case, will repeatedly output the same findings.

If you want to ignore something, remove any memory files from the repo before running the model, and put a very general term in the initial prompt to avoid the whole XYZ class of vulnerabilities.

But be careful with too many negations in the prompt, because the model may do exactly what you're trying to avoid after a few rounds of context compaction.

## Context is everything

Understanding the context window is the most crucial aspect of using coding agents. There is a never-ending balance between giving the model enough context to make good output and not overwhelming it with too much information. To see how much context is being used, Claude has a built-in command `/context`, which shows the current context window.

![context](/web/Screenshot%20From%202026-03-11%2023-46-58.png)

Every [MCP](https://modelcontextprotocol.io/docs/getting-started/intro), [skill](https://agentskills.io/home), prompt, and chat history takes up space in the context window, which gradually worsens the agent until automatic compact or clear commands are used to reduce the context size. This might delete important information that you added earlier; however, the model did not consider it important enough to keep.

Even if you told the model something, it might still use wrong part of the memory for backing up its claims, leading to bad outputs. This is not often problem of the model because it reasons based on memory, but memory pollution is a real issue.

### Tokens

For tools that are likely included in the training data, like AFL++ or nmap, the model probably already knows how to use them, so adding MCP or skill makes it worse.

Try not to use skills or MCPs unless you have a good reason.

By the way, using less popular languages, like Haskell or Czech, will result in more tokens, since the tokenizer optimizes for the most common patterns in the training data.

For example, Python code translates into 6 tokens:

![python_tokens](/web/Screenshot%20From%202026-03-11%2023-07-04.png)

The Haskell equivalent translates into more tokens, even though it's shorter:

![haskell_tokens](/web/Screenshot%20From%202026-03-11%2023-08-45.png)

Context has an impact on token usage, so it literally means you pay for using Czech instead of English.

## Models

I tried ChatGPT, Claude, and other models, but none of them could outperform Opus 4.6, not even newer GPT 5.4. I am not saying the models are better or worse; I am only saying that Opus was best for vulnerability research.

However, not all Opuses are equal; for example, the Opus in [Antigravity](https://antigravity.google/) is much stupider than the One in Claude Code.

I heard that success of Opus can be attributed to Anthropic's focus on building very good with most basic tooling like bash and sed. Instead of trying to make a cheaper model more usable with fancy tooling like [vector indexing](https://cursor.com/docs/agent/tools/search).

However reality might be more complex, and only time will tell.

BTW for searching information, Gemini in web client with "deep research" is much better than any alternatives even though gemini is considered to be less capable model compared to others.

I guess it makes sense that Google as company makes nice search tools.

## Vulnerabilities

From my experience, AIs detect only well documented and known classes of vulnerabilities like OWASP 10 or memory corruption issues. They worked really well on classes like SSRF, XSS and buffer overflows.

They struggle with vulnerabilities that require more creativity.

For example I had to manually guide the model to find specific research on particular vulnerabilities like exfiltration of data with CSS fonts.

This could be because the techniques are niche and fairly new, however researcher must keep in mind, that although AI are trained on vast amounts of data, they still do not see attack vectors like human would.

## Cost and manual overhead

I played with GLM 4.7 for agentic small tasks, and created guiderails for this weaker but cheap model; however, when I look back. The time invested in fixing the issues exceeded the cost of using the more expensive model.

And bear in mind that every guiderail you build will eventually be outdated when smarter models get cheaper, do not forget to include time when calculating price :D.

This goes without saying but the manual overhead is still tremendous, you need to constantly monitor and adjust tasks, it is not an autonomous process at all.

## Conclusion

In conclusion, while models like Claude currently excel at some aspects of vulnerability research, they are still limited in their ability to understand context and threat model of specific projects. Do not rely on them too much, however completely not using AI is not a viable option either.

## References

- [Defeating Nondeterminism in LLM Inference](https://thinkingmachines.ai/blog/defeating-nondeterminism-in-llm-inference/)
- [Evaluating AGENTS.md: Are Repository-Level Context Files Helpful for Coding Agents?](https://arxiv.org/pdf/2602.11988)
- [AGENTS.md Manifesto](https://agents.md/)
- [Skills](https://agentskills.io/home)
- [OpenAI Tokenizer](https://platform.openai.com/tokenizer)