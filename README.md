# README 2 of 3: SDDs from AI Loops
>Creating Software Development Documents

This README describes a **SKILL** for **Hermes Agent** that uses two (2) **Looping Agents** to create, and refine, **Software Development Documents** (SDDs that are later used to build deployable software).

---

## Requirements

*   Ubuntu Desktop 22.04 LTS or later

*   git `sudo apt install -y git-all`

*   curl `sudo apt install -y curl`

*   Hermes Agent - Modern Link `curl -fsSL https://hermes-agent.nousresearch.com/install.sh | bash`

*   Hermes Agent - Classic Link `curl -fsSL https://raw.githubusercontent.com/NousResearch/hermes-agent/main/scripts/install.sh | bash`

---

## TL;DR Simple Installation Process

Here is the Hermes Agent SKILL for creating Spec files:

`/plan`

Here are the [Hermes Agent installation directions](./Install_Prompt_Directions_v0.5.6.md).

Here are the two (2) Hermes Agent commands for starting this SKILL:

*   `/dc`: An acronym for Dual Coders.

*   `/dcv`: Activates Verbose Mode.

*   Example: `/dcv /path/to/software_spec.md` runs the SKILL in Verbose Mode so that all of the operations can be observed by the Hermes Agent User.

---

## Detailed Installation Process

While the previous `TL;DR` section is all you need to "spin up" a practical deployment, the following sections will help anyone interested in modifying this repo so that the mod can work with other AI agents.

### Clone the Repo

*   In the terminal, I change to the Downloads directory:

```bash
cd ~/Downloads
```

*   I run the following command to clone this repo:

```bash
git clone https://github.com/DigitalCoreNZ/2_of_3_SDDs_from_Loops
```

---

### Commands for Hermes Agent

*   I update Hermes Agent:

```bash
hermes update
```

*   I connect Hermes Agent to an AI model of my choice:

```bash
hermes model
```

*   I run Hermes Agent:

```bash
hermes
```

*   I run [this prompt within Hermes Agent](Install_Prompt_Directions_v0.5.6.md) to install the SKILL.

*   Here are the two (2) Hermes Agent commands for this SKILL:

> `/dc`: An acronym for Dual Coders.
>
> `/dcv`: Activates Verbose Mode.
>
> Example: `/dcv /path/to/software_spec.md`.

---

## License

Due to Hermes Agent being released under the MIT license, this project is also released under the same, permissive requirements, i.e. no restrictions, no limits, no liability. This means that you are welcome, and even encouraged, to use the content of this repository as the foundation for derivative works. For instance, if you are inspired to fork this repo and adapt it to your favourite AI agent, then send me an email with the subject line `SDDs-from-Loops` (because I already have an email rule setup) and a link to your project so I can add it to the `Adaptations` section below.

---

## Adaptations

None at this time.

---

## A Complete Breakdown

There is a report called `[SDDs_from_AI_Loops_v0.5.6.md](./SDDs_from_AI_Loops_v0.5.6.md)`. This file contains _every painful detail_ that relates to the creation of this SKILL.

---

## The Hermes Agent Installation Prompt

There is another file called `[Install_Prompt_Directions_v0.5.6.md](./Install_Prompt_Directions_v0.5.6.md)` that contains the Hermes Agent installation prompt.

---

## Versions

The first three (3) versions of this project were low-effort, low-reward internal experiments. Version v0.5.4, however, introduced Role Definitions, version v0.5.5 included Watch Mode and Control Mode, and this version (v0.5.6) includes Recovery Mechanisms.

Once I confirm the stability of this version (by using it to build a whole range of computer programs and utilities, and then testing the results), I will replace the internal-versioning system (v0.5.1, v0.5.2, v0.5.3,...) with a git-enabled system, and test it against a local instance of GitLab CE (a version control service where I can test the SKILL against CI/CD pipelines and runners).

Other features for future versions include queues, sandboxing, suspend and resume, durable memory, and event streams.

Eventually, I will use the accumulated features of this SKILL to build a software development app that includes:

* An API,
* The [fifth specification release of MCP from Anthropic](https://claude.com/blog/bringing-mcp-2026-07-28-to-claude),
* Support for locally-run GGUF AI models from Hugging Face, and
* Versions for Linux (.AppImage), macOS (.app), and Windows (.exe).

---

## Conclusion: README

This is the first Hermes Agent SKILL I have ever released into the wild.

My objective is to combine the underlying concepts of this Hermes Agent SKILL with the [Spec Files from Loops](https://github.com/DigitalCoreNZ/1_of_3_Specs_from_AI_Loops) repo and the [Apps from AI Loops](https://github.com/DigitalCoreNZ/3_of_3_Apps_from_AI_Loops) repo, resulting in a complete Hermes Agent workflow that can be applied to the software development app that was mentioned in the previous section.

Thank-you for reading until the end of this file.

As mentioned above, this is the end of the file.

You can leave as I have nothing left to say.

Go and do something useful.

Bye-bye now.