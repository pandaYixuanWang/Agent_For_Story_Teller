# Hippocratic AI Coding Assignment
Welcome to the [Hippocratic AI](https://www.hippocraticai.com) coding assignment

## Instructions
The attached code is a simple python script skeleton. Your goal is to take any simple bedtime story request and use prompting to tell a story appropriate for ages 5 to 10.
- Incorporate a LLM judge to improve the quality of the story
- Provide a block diagram of the system you create that illustrates the flow of the prompts and the interaction between judge, storyteller, user, and any other components you add
- Do not change the openAI model that is being used. 
- Please use your own openAI key, but do not include it in your final submission.
- Otherwise, you may change any code you like or add any files

---

## Rules
- This assignment is open-ended
- You may use any resources you like with the following restrictions
   - They must be resources that would be available to you if you worked here (so no other humans, no closed AIs, no unlicensed code, etc.)
   - Allowed resources include but not limited to Stack overflow, random blogs, chatGPT et al
   - You have to be able to explain how the code works, even if chatGPT wrote it
- DO NOT PUSH THE API KEY TO GITHUB. OpenAI will automatically delete it

---

## What does "tell a story" mean?
It should be appropriate for ages 5-10. Other than that it's up to you. Here are some ideas to help get the brain-juices flowing!
- Use story arcs to tell better stories
- Allow the user to provide feedback or request changes
- Categorize the request and use a tailored generation strategy for each category

---

## How will I be evaluated
Good question. We want to know the following:
- The efficacy of the system you design to create a good story
- Are you comfortable using and writing a python script
- What kinds of prompting strategies and agent design strategies do you use
- Are the stories your tool creates good?
- Can you understand and deconstruct a problem
- Can you operate in an open-ended environment
- Can you surprise us

---

## Other FAQs
- How long should I spend on this? 
No more than 2-3 hours
- Can I change what the input is? 
Sure
- How long should the story be?
You decide

---

## My Solution

### Architecture

This system implements a **multi-agent draft-judge-refine loop** built on top of `gpt-3.5-turbo`:

```
User Input → Storyteller (Draft) → Judge → [APPROVED] → Output
                    ↑                  |
                    └── Storyteller ←──┘ [REVISION_NEEDED, up to 3 iterations]
                        (Refine)
```

Three specialised agents collaborate, each given a carefully engineered system prompt:

| Agent | Role |
|---|---|
| **Storyteller (Draft Mode)** | Writes the initial bedtime story from the user's request |
| **Judge** | Scores the draft 0–10 across 7 rubric criteria and returns `APPROVED` or `REVISION_NEEDED` with specific, actionable feedback |
| **Storyteller (Refine Mode)** | Revises the draft using the Judge's critique while preserving the original premise |
| **Main Controller** | Orchestrates the loop, enforces a score guardrail (≥ 8/10 required), and caps iterations at 3 |

The system block diagram is in [`diagram.md`](diagram.md).

### Key Design Decisions

- **Score guardrail**: even if the Judge outputs `APPROVED`, the Main Controller re-checks the numeric score and forces `REVISION_NEEDED` if it is below 8/10 — preventing a lenient model from bypassing the quality bar.
- **Prompt injection defence**: user input is wrapped in `<user_request>` XML tags with an explicit framing instruction, reducing the risk of prompt injection overriding the system prompt.
- **Separate draft vs. refine prompts**: the draft prompt optimises for imagination and tone; the refine prompt specifically instructs the model to address the judge's critique without losing the original charm.
- **Judge rubric**: seven criteria (alignment, age appropriateness, bedtime tone, engagement, structure, emotional resolution, length) keep feedback concrete and consistent.

### Setup

1. **Clone the repo**
   ```bash
   git clone <repo-url>
   cd Agent_For_Story_Teller
   ```

2. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

3. **Set your OpenAI API key** — create a `.env` file in the project root:
   ```
   OPENAI_API_KEY=sk-...
   ```
   > ⚠️ Never commit this file. It is already in `.gitignore`.

### Usage

**User-Facing Mode** (default) — prints only the final polished story:
```bash
python main.py
```

**Demo / Debug Mode** — shows the full agentic loop (drafts, judge scores, revisions):
```bash
python main.py --debug
```

### Example

```
$ python main.py
Welcome to the AI Storyteller System!
What kind of story do you want to hear? A brave little snail who wants to climb a mountain

========================================
FINAL STORY:
========================================
# The Brave Little Snail

...
```