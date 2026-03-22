---
name: ai-ml
description: "AI & Machine Learning. (Mandatory Agentic Multi-Agent Orchestration, Zero-Interruption Policy for Autonomous Agents, Hard-Binding Local Deterministic Tools to AI Logic, Constraint Pruning for High-Reasoning Models, Distinguishing Behavioral Policy from Tool Capability) Use when user mentions: LLM, GPT, Claude, Gemini, prompt, model, training, inference, embedding."
---

# AI & Machine Learning

Conversations: 1 | Patterns: 5

## 1. Mandatory Agentic Multi-Agent Orchestration

Enforcing a transition from planning to a specialized multi-agent 'team' execution, even for simple tasks, to ensure workflow consistency and leverage parallel processing capabilities that the AI might otherwise skip for perceived efficiency.

### Steps

1. Define a mandatory 'Plan then Team' execution rule in the agent's core instruction file
2. Explicitly override the AI's internal heuristics that favor direct, single-agent execution for small tasks
3. Assign specific sub-tasks to parallel agents to maintain the architectural pattern of the project


## 2. Zero-Interruption Policy for Autonomous Agents

Overriding the default 'ask-if-unsure' behavior of LLMs to enable truly autonomous operation. This is critical in the AI domain where conversational friction can halt automated pipelines and background tasks.

### Steps

1. Identify conversational triggers where the model is likely to pause for user confirmation
2. Implement a 'No Interruption' rule in the agent's behavior instructions to force independent decision-making
3. Establish clear priorities or fallback mechanisms so the agent can proceed without human input


## 3. Hard-Binding Local Deterministic Tools to AI Logic

Integrating specific local infrastructure and deterministic analysis tools (like custom linters or review scripts) directly into the AI's verification loop to override its probabilistic and often less rigorous self-review capabilities.

### Steps

1. Define exact file paths and execution commands for local verification tools in the instruction set
2. Direct the agent to rely on the output of these deterministic tools rather than its own reasoning for the final validation
3. Enforce the use of these tools as a mandatory gateway before task completion


## 4. Constraint Pruning for High-Reasoning Models

Removing redundant or common-sense instructions from agent prompts to avoid 'over-constraining' advanced models. Excessive instructions act as noise and can degrade the performance of highly capable AI agents.

### Steps

1. Audit existing instructions for overlap with the model's inherent reasoning and default behaviors
2. Remove 'shackle' constraints that repeat standard practices the model already follows
3. Condense instructions to focus strictly on project-specific overrides and unique workflow requirements


## 5. Distinguishing Behavioral Policy from Tool Capability

Strategically separating process enforcement (Policy) from tool availability (Capability). While plugins provide the means to perform tasks, a central policy file is required to mandate their use and define the specific team-based workflows.

### Steps

1. Identify the difference between what a tool 'can' do (plugin) and what an agent 'must' do (policy)
2. Use central configuration files to enforce specific sequences and tool combinations
3. Evaluate tool-instruction overlap to ensure instructions only cover enforcement that tools alone cannot provide



