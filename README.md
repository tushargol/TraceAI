# TrustTrace AI

A research project exploring how to make AI agents safer by tracking where their instructions come from and what they're actually allowed to do.

## The Problem

AI agents are getting really good at using tools to read emails, browse the web, and work with files. But there's a catch: when they process external content, that content might contain hidden instructions that hijack the agent. Think about it like this - you ask an agent to summarize your emails, but one email contains a sneaky command telling the agent to send all your confidential files to an attacker.

Existing solutions like RBAC and tool permissions help, but they don't fully solve this problem. They're good at saying "you can use this tool" but not great at saying "you can use this tool for this specific purpose, with this specific data, from this specific source."

## What I'm Building

TrustTrace combines a few ideas:
- **Provenance tracking**: Keeping track of where every piece of information came from
- **Intent-based authorization**: Figuring out what the user actually wants the agent to do, then only allowing actions that match that intent
- **Deterministic enforcement**: Making security decisions outside the AI model, so even if the model gets tricked, the security still holds

The big question I'm trying to answer: does tracking where instructions come from actually help prevent attacks, compared to just giving agents a set of allowed tools?

## How It Works

### Agent Simulator
A simple AI agent that can read synthetic emails and documents, with tools for searching files, summarizing content, and taking actions. Every tool call goes through an authorization check before it actually executes.

### Provenance Engine
This tracks where every piece of information comes from - was it from the user? From the system? From an external email or document? It builds a graph showing how information flows through the system, so we can see if an action was influenced by untrusted content.

### Authorization Engine
This takes what the user asked for and figures out what the agent should actually be allowed to do. It converts "summarize my emails" into something like "you can read emails and you can summarize, but you can't send anything to external addresses." Then it enforces those rules deterministically, outside the AI model.

### Risk Analysis (Optional)
A lightweight ML model that can flag suspicious action proposals. This is supplementary - the real security comes from the deterministic checks, but this can help catch edge cases.

### Dashboard
A way to visualize what's happening - what the agent is trying to do, what got blocked, and why. Good for debugging and understanding the system.

## Tech Stack

I'm keeping it simple:
- Python 3.11+ with FastAPI for the API
- A custom ReAct-style agent loop (no heavy frameworks initially)
- NetworkX for the provenance graph
- SQLite for data storage (can upgrade to PostgreSQL later)
- pytest for testing
- Streamlit for the dashboard (starting with CLI/logging first)

## Development Plan

### Phase 1: Research (Done)
I started by reviewing what's already out there - things like FIDES, PAuth, and IBAC. Figured out where the gaps are and what's actually novel about combining provenance tracking with intent-based authorization. Also defined the threat model and what I'm actually trying to test.

### Phase 2: Architecture (Done)
Designed the component architecture, defined the data structures, and documented the threat model and security boundaries. Created the project structure and initial documentation.

### Phase 3: MVP Implementation
Building the synthetic environment with sample emails and files, implementing a simple agent with tool calling, and adding the authorization engine with intent parsing. Writing tests as I go.

### Phase 4: Provenance Tracking
Building the provenance graph with NetworkX, figuring out how labels should propagate through operations, and testing multi-step workflows.

### Phase 5: Testing Against Attacks
Creating a benchmark with about 50 test cases (mix of benign and adversarial), running the different configurations, and seeing what actually gets blocked.

### Phase 6: AI-Assisted Analysis (Maybe)
If the deterministic enforcement works well, I might add a lightweight ML model to help flag suspicious actions. But only if it actually adds value.

### Phase 7: Dashboard and Deployment
Building a simple dashboard to visualize what's happening, adding proper logging, and maybe containerizing it.

### Phase 8: Wrap Up
Writing up the technical report, documenting everything, making sure experiments are reproducible, and figuring out what worked and what didn't.

## Project Structure

```
trusttrace-ai/
├── app/
│   ├── agent/           # Agent simulator
│   ├── provenance/      # Provenance tracking engine
│   ├── authorization/   # Authorization engine
│   ├── risk_analysis/   # AI-assisted risk analysis (optional)
│   ├── tools/           # Tool implementations
│   ├── api/             # FastAPI endpoints
│   └── dashboard/       # Streamlit dashboard
├── policies/            # Authorization policies
├── data/
│   ├── synthetic/        # Synthetic test data
│   └── benchmark/       # Benchmark test cases
├── experiments/         # Experiment scripts and results
├── tests/
│   ├── unit/            # Unit tests
│   ├── integration/     # Integration tests
│   └── adversarial/     # Adversarial test cases
├── docs/
│   ├── architecture.md  # Architecture documentation
│   ├── threat_model.md  # Threat model documentation
│   └── research_design.md # Research design documentation
├── scripts/             # Utility scripts
├── Dockerfile           # Docker configuration (optional)
├── requirements.txt     # Python dependencies
├── pyproject.toml      # Project configuration (optional)
└── README.md           # This file
```

## Security Model

### What I'm Protecting Against
- Direct and indirect prompt injection (hidden commands in emails, docs, web pages)
- Unauthorized file access or data transfers
- Multi-step manipulation where an attacker gradually expands what the agent can do
- Permission escalation
- Attempts to bypass the security layer

### What I Trust
- The authorization engine (deterministic policy enforcement)
- The provenance tracking system (graph construction and label propagation)
- The user interface (authentic user interaction)
- TLS-encrypted communication
- The synthetic environment (since I control it)

### What I Don't Trust
- The LLM agent itself (it can be manipulated)
- External content (emails, documents, web pages - assume they're all potentially malicious)
- Agent memory (might be poisoned)
- Tool parameters generated by the agent

## How I'm Measuring Success

### Security Metrics
- How often unauthorized actions get attempted
- How many attacks actually succeed
- How well the system detects when someone tries to expand beyond their intended scope

### Utility Metrics
- How often legitimate tasks actually complete successfully
- How often the system incorrectly blocks legitimate actions (false positives)
- How much latency the security checks add

### What I'm Comparing
I'm testing four different setups:
1. No protections at all (baseline)
2. Just tool-level permissions (like RBAC)
3. Intent-based authorization only
4. The full TrustTrace system (intent + provenance)

## Getting Started

```bash
# Clone the repo
git clone <repository-url>
cd trusttrace-ai

# Set up a virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt
```

## Running It

```bash
# Run the agent simulator
python -m app.agent.main

# Run the benchmark evaluation
python -m experiments.run_benchmark

# Run tests
pytest tests/

# Start the dashboard (when I get around to implementing it)
streamlit run app/dashboard/main.py
```

## Contributing

This is a research project, so contributions should align with the research goals and keep experiments reproducible. That said, if you have ideas or find bugs, feel free to open an issue or PR.

## License

To be determined - I'll figure this out as the project matures.

## Acknowledgments

This project builds on existing research from:
- InjecAgent: Benchmarking Indirect Prompt Injections
- FIDES: Information-Flow Control for AI Agents
- PAuth: Task-Scoped Authorization via Natural Language Slices
- IBAC: Intent-Based Access Control
- OWASP AI Agent Security Cheat Sheet
