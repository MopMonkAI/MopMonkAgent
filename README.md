# MopMonk Agent

> Memory-Centric Agent Design for Automated Vulnerability Discovery

This report presents **MopMonk Agent**, a memory-centric Multi-Agent design for general-purpose vulnerability mining tasks. Its core idea is to continuously organize code observations, negative evidence, candidate inputs, and verification feedback into structured vulnerability memory, allowing later exploration to converge from accumulated evidence instead of repeatedly restarting trial and error from scratch.

## Contact
MopMonk.AI@proton.me

## Benchmark

We evaluated MopMonk on CyberGym Level 1 with a 4-hour timeout setting. CyberGym Level 1 focuses on the automated reproduction of real-world software vulnerabilities. Each task provides a vulnerability description and an unpatched codebase. The goal is not merely to identify a plausible issue, but to generate a working PoC input that triggers the target vulnerability on the vulnerable version and no longer triggers it on the fixed version. In other words, CyberGym measures the full closed loop from vulnerability clues, code understanding, and input construction to execution-based verification.

## Base Model

This submission uses **MiniMax M3** as the base model. We chose MiniMax M3 because it combines long-context capacity, the effective capacity of an MoE architecture, efficient sparse attention, strong coding ability, and stable execution in long-horizon agentic and cowork settings. For automated vulnerability mining tasks, the base model contributes most directly through long-context understanding, code reasoning, tool-feedback absorption, and coherence over extended task loops.

## Core Design

### Vulnerability-Oriented Memory Design

Our core contribution is a structured memory method designed for vulnerability mining tasks. Rather than simply storing chat history or relying on long context alone, it organizes vulnerability goals, code paths, input formats, negative evidence, execution feedback, and verification state into continuously updated task memory.

This memory method organizes information around key objects in the vulnerability mining process:

- **Vulnerability-goal memory**: Captures the target vulnerability, success conditions, acceptable verification standards, and the most important constraints for the current task.
- **Code-path memory**: Captures confirmed entry points, harnesses, parsing chains, suspicious functions, and key data flows, so later attempts do not need to rediscover them.
- **Input-format memory**: Captures file formats, field relationships, length constraints, boundary conditions, and known valid or invalid input patterns.
- **Candidate-PoC memory**: Captures candidate inputs, generation rationale, triggering behavior, mutation direction, and hypotheses that still require verification.
- **Negative-evidence memory**: Captures non-triggering attempts, unreachable paths, build failures, format errors, and other negative results to avoid repeated search.
- **Verification-state memory**: Tracks whether a candidate PoC triggers a crash and records why it fails when it does not trigger.
- **Next-constraint memory**: Instead of producing an abstract plan, it extracts constraints that the next try must satisfy, such as a branch to reach, a field to adjust, or a failure cause to avoid.

This memory design turns vulnerability mining from "read the context again and guess again" into an evidence-based convergence process. Every code-reading step, execution result, failed submission, and verification response is converted into reusable constraints for future PoC generation.

### Memory-Driven Vulnerability Mining

In vulnerability mining tasks, the system first initializes vulnerability memory by scanning the codebase and using candidate triggering paths and directory information as the starting point for planning. It then advances step by step, attempting to converge toward the concrete code location that triggers the crash. After that, each exploration attempt reads the current memory, tests one specific hypothesis, and writes the result back into memory.

This mechanism brings several direct effects:

- **Less repeated exploration**: Failed paths are explicitly recorded, so later attempts do not keep following the same source path or input mutation direction.
- **Preserved negative evidence**: Non-triggering attempts, malformed inputs, and unreachable paths become constraints instead of being discarded.
- **Faster PoC convergence**: Each candidate PoC mutation can inherit previously discovered code paths, input-format constraints, and verification feedback.
- **Lower long-context burden**: The model does not need to understand the entire task from scratch every time; it reads the most relevant evidence from structured vulnerability memory.
- **Higher useful-experiment density within the budget**: To control practical operating cost, each exploration attempt has a time limit. Under this constraint, the memory method helps the system spend more time on new and useful hypotheses.

### Multi-Agent Parallel Exploration Under Shared Memory

The system can launch multiple exploration attempts on top of the same vulnerability memory. Each attempt reads the current memory, verifies a concrete hypothesis, and writes the result back as new evidence or constraints. Parallelism is not the core method itself, but a way of applying the memory method: multiple Agents share the same vulnerability memory, so they can inherit one another's failed attempts, candidate PoCs, and verification results.

For the same vulnerability description, there may be multiple reasonable directions, such as inferring from crash stacks or sanitizer types, or constructing minimal inputs from boundary conditions. Shared memory allows these directions to progress in parallel while avoiding repeated work.

## Contribution

- **Base-model capability provides the search foundation**: MiniMax M3 provides long-context understanding, code reasoning, and long-horizon execution ability, enabling the system to handle large codebases, multi-round execution feedback, and complex PoC construction.
- **Vulnerability memory provides the convergence mechanism**: Structured memory unifies useful leads, negative evidence, and verification state into one task state, turning PoC search from open-ended trial and error into evidence-based convergence.
- **Shared-memory exploration improves cost efficiency**: Multiple exploration attempts share the same vulnerability memory, cover more plausible paths within the exploration budget, and avoid repeated failed directions.

Overall, our main contribution is to turn vulnerability mining into an accumulative, constraint-driven, and verifiable memory-update process. MiniMax M3 provides strong base-model capability, while structured vulnerability memory continuously steers that capability toward more effective directions.

## Main Result

| Rank | Agent | Model | Success Rate | Date | Source |
|---:|---|---|---:|---|---|
| 1 | Crystalline | Claude Opus 4.6 | 89.6% | 2026-06-08 | Independent researcher |
| 2 | MDASH | Multi-model | 88.4% | 2026-05-12 | Microsoft |
| 3 | OpenAI Agent | GPT-5.5-Cyber | 85.6% | 2026-06-22 | OpenAI |
| 4 | Anthropic Agent | Claude Mythos Preview | 83.1% | 2026-04-07 | Anthropic |
| 5 | OpenAI Agent | GPT-5.5 | 81.8% | 2026-04-23 | OpenAI |
| 6 | OpenAI Agent | GPT-5.4 | 79.0% | 2026-04-23 | OpenAI |
| 7 | MopMonk Agent | MiniMax M3 | 73.1% | 2026-06-29 | MopMonk Agent |

## Limitations

- MopMonk Agent remains a closed-source system.
- The current system still requires substantial computational resources, and further optimization is needed to improve its overall efficiency.
