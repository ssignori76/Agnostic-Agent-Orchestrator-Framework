# Agnostic Agent Orchestrator Framework (AAOF)

An AI-driven project management framework designed to automate the deployment and maintenance of complex IT environments (3-tier apps, system integrations, lab demos) through a structured, document-based workflow.

## 🚀 The Concept
The **Agnostic Agent Orchestrator Framework** is built for non-programmers who need to delegate the heavy lifting of environment setup and code development to AI Agents. Unlike standard prompts, this framework provides a **structured OS** for the AI, ensuring:
* **Tech-Agnosticism:** Works with any stack (Java, Python, PHP, Docker, K8s).
* **State Persistence:** Uses physical JSON files to maintain session memory across different AI interactions.
* **Production Standards:** Enforces modular code, automated backups, and detailed changelogs.

## 🏗 Framework Structure
The project is organized to separate "Rules" from "Data":
- `agent.md`: The core logic and workflow the AI must follow.
- `config.json`: Your technical desires (stack, target environment).
- `/rules`: Markdown files defining how the code should be written and secured.
- `/specs`: Functional requirements for the AI to implement.
- `/output`: The final product (code, containers, manifests) and the "Source of Truth" (`deployed_state.json`).
- `/backups`: Every change is snapshotted for easy rollback.

## 🔄 How It Works
1. **Configure:** Define your stack in `config.json`.
2. **Specify:** Drop your requirements in `specs/active/`.
3. **Run:** Feed `agent.md` to your AI Agent (Claude, GPT, Gemini).
4. **Approve:** Review the AI's execution plan.
5. **Deploy:** Get your ready-to-run code and containers in the `output/` folder.

## 🛠 Key Features
- **Modular by Design:** Enforces small, maintainable classes (max 150 lines).
- **Self-Documenting:** Automatically updates a `changelog.md` and a technical `deployed_state.json`.
- **Safe Evolutions:** Handles project updates by reading existing states before making changes.
- **Local Lab Ready:** Native support for `Docker Compose` and `Minikube` (Kubernetes).

---
*Created with 💡 to empower non-developers in managing professional AI-driven project lifecycles.*
