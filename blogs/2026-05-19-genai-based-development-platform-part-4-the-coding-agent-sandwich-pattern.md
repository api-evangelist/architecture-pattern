---
title: 'GenAI-based development platform - part 4: The coding agent sandwich pattern'
url: http://microservices.io//post/architecture/2026/05/19/genai-development-platform-part-4-coding-agent-sandwich-pattern.html
date: '2026-05-19'
author: ''
feed_url: https://microservices.io/feed.xml
---
This article describes the “coding agent sandwich” — an architecture pattern consisting of a tasty filling of LLM invocations sandwiched between two slices of plain old code (POC). This article is part of a series about the GenAI-based development platform (a.k.a. harness) that I’ve been developing to make GenAI-based coding agents like Claude Code more productive, more secure and less frustrating.
The complete list of articles in the series is as follows: Part 0 - My GenAI development workflow: Idea to Code Part 1 - Guardrails for GenAI coding agents Part 2 - How Idea to Code turns an idea into working, tested software Part 3 - Announcing Isolarium, three flavors of secure sandboxes for GenAI-based coding agents Part 4 - The coding agent sandwich pattern The worked example is the implement plan sub-workflow , which is the last step of idea-to-code , my open-source GenAI-based development workflow .
The implement plan workflow uses test-driven development (TDD) to turn the generated plan into code. Before looking at the different parts of the sandwich, let’s start with an overview of the implement plan workflow. An overview of the implement plan workflow The input to this workflow is the plan.
The plan consists of a set of tasks.
The implement plan workflow turns each task into a Git commit on either trunk or a pull request branch.
The following diagram shows the implement plan workflow: The workflow begins with i2code cleaning up from a previously failed implementation by committing any uncommitted changes and pushing them to the remote repository. Once it has cleaned up, i2code iteratively implements each task in the plan.
The plan implementation loop consists of the following steps: If there’s a running CI build, wait for it to complete. If the CI build failed, fix it. Respond to any comments on the pull request. Get the next task from the plan. If there are no more tasks, break the loop. Implement the task. Mark the task as complete in the plan. Commit the changes to Git. Push the commit to the remote repository. Why not implement the implement plan workflow using a coding agent? In theory, i2code could simply implement the implement plan workflow by invoking a coding agent (e.g. Claude Code) with a suitable prompt that tells it to implement the entire plan: claude -p "Implement the following plan: ${plan_file} using the following workflow:

commitChanges()
pushCommits()
while True
    1. If there's a running CI build, wait for it to complete.
    2. If the CI build failed, fix it.
    3. Respond to any comments on the pull request.
    4. Get the next task from the plan.
    5. If there are no more tasks, break the loop.
    6. Implement the task.
    7. Mark the task as complete in the plan.
    8. Commit the changes to Git.
    9. Push the commit to the remote repository.
" However, in practice, this approach is often unreliable and it’s certainly inefficient.
The non-deterministic nature of LLMs means that it’s unlikely that the agent will successfully implement the entire plan in one go.
For example, I’ve found that it will simply stop partway through.
Also, LLMs are computationally expensive, so it’s wasteful to use an LLM when there is a far more efficient way to implement a function: plain old code (POC) - the top slice of the sandwich. The top slice: orchestrating the implement plan workflow using plain old code i2code implements the implement plan workflow using plain old code.
The actual Python code is not that different from the pseudo-code in the workflow diagram above: def execute ( self ): self . _loop_steps . commit_recovery . commit_if_needed () if self . _git_repo . has_unpushed_commits (): self . _push_and_ensure_pr () while True : if self . _loop_steps . build_fixer . check_and_fix_ci (): ... This code is deterministic, efficient and easy to develop and debug.
There’s no need to experiment with different prompts and hope that the agent will do the right thing only to find that it doesn’t.
Don’t waste your time trying to solve a problem with an LLM when there are far simpler solutions. Let’s now look at the sandwich’s filling: the coding agent. The filling: invoking the coding agent for narrowly defined actions Of course, there are some steps in the workflow that are not easily implemented using POC.
For those, the POC-based orchestrator invokes the coding agent for narrowly defined tasks that cannot be implemented deterministically.
Specifically, the i2code implement workflow invokes the coding agent for the following tasks: Committing changes - completing the pre-commit checklist , generating a commit message and committing the changes Fixing a broken CI build - analyzing the CI failure, identifying the cause and creating a commit that fixes it Responding to pull request comments - analyzing comments, grouping related comments, and generating commits Implementing a task - analyzing the task, identifying the necessary code changes, making the changes that implement the task, marking the task as complete and creating a commit for the changes This approach allows us to leverage the strengths of both POC and the coding agent.
However, there’s still one more layer of the sandwich: the deterministic tools that the coding agent can use to implement the tasks. The bottom slice: deterministic tools used by the coding agent To complete these tasks, the coding agent must be grounded in reality and perform various deterministic actions, such as running tests, verifying code quality, committing and pushing changes, and monitoring CI builds. For example, during the implement plan workflow, the coding agent invokes various tools including: uv or gradlew commands for running tests git commands gh commands for diagnosing a failed build i2code plan commands for marking a task in a plan file as complete, etc. CodeScene MCP server for analyzing code quality and identifying problems gitleaks for checking for secrets in the code before committing Claude Code has either built-in knowledge of these tools or is instructed to use them via skills or CLAUDE.md instructions. We’ve now seen all three slices. 
Let’s step back and look at what kind of pattern this is. Functional decomposition, layered and fractal This approach of using a mixture of POC and LLMs is an example of functional decomposition - a general software design principle where a system is divided into components based on responsibility, and each function is implemented using the most appropriate technology. In this case: deterministic, well-defined logic is implemented using POC ambiguous, language-oriented, or probabilistic tasks are implemented using LLMs Rather than treating the LLM as the entire application, the system decomposes responsibilities and uses the LLM only where its capabilities provide value. Functional decomposition can be applied recursively.
The tools that form the bottom slice are all deterministic, but in principle a tool could itself incorporate LLM-based technology - in which case the tool is its own sandwich of POC orchestrating LLM calls.
The architecture is fractal as well as layered. Use POC whenever you can, LLMs only when you must The practical rule: reach for POC first, fall back to an LLM only where determinism breaks down, and give the LLM deterministic tools so that it’s grounded in reality. Need help with modernizing your architecture? I help organizations modernize safely and avoid creating a modern legacy system — a new architecture with the same old problems. If you’re planning or struggling with a modernization effort, I can help. Learn more about my modernization and architecture advisory work →
