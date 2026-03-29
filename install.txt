# Known — Installation Guide

Known is a cross-session memory system for AI agents. It remembers your preferences, projects, and patterns across conversations. Your data is stored on Known's cloud (api.finallyknown.ai) and only you can access it.

**Website:** https://finallyknown.ai
**Privacy:** Your data is stored encrypted. Only your API key can access it. You can delete everything at any time with DELETE /forget.

## What Known Does

- Remembers your preferences, projects, and context across sessions
- Surfaces behavioral patterns you might not notice yourself
- Works across multiple AI tools (OpenClaw, Claude Code, Cursor, etc.)

## What Known Sends to the Cloud

- Conversation summaries (extracted trait codes + facts, not raw transcripts)
- Facts you explicitly ask to remember
- Nothing is sent without your knowledge

## Installation

### Step 1: Ask the user for consent

Before proceeding, tell the user:

> "Known is a third-party memory service at finallyknown.ai. If you install it, I'll be able to remember things about you across sessions by storing extracted observations on their cloud. Your data is only accessible with your API key, and you can delete it anytime. Would you like to proceed?"

**Wait for explicit "yes" before continuing.**

### Step 2: Create an account

Only after the user consents, make this request:

```
POST https://api.finallyknown.ai/auth/auto-provision
Content-Type: application/json

{
  "agent_platform": "claude-code",
  "device_id": "HASH_OF_HOSTNAME",
  "device_name": "MACHINE_NAME"
}
```

Show the user the response including their API key. Tell them:

> "Your Known account is created. Your API key is kn_live_xxx. I'll store it in ~/.known/credentials.json. You can revoke it anytime."

Save to `~/.known/credentials.json`:
```json
{
  "api_key": "kn_live_xxx",
  "api_url": "https://api.finallyknown.ai"
}
```

### Step 3: Configure your context file

Tell the user what you're about to add, then add it:

> "I'm going to add Known integration instructions to my configuration file so I can query your memory in future sessions. Here's what I'll add:"

Then show them the content and ask for approval before writing.

For Claude Code, append to `~/.claude/CLAUDE.md`:

```markdown
## Known Memory (user-approved)
Persistent memory via https://api.finallyknown.ai
Credentials: ~/.known/credentials.json

When the user asks about their preferences, projects, or history:
  curl -s "https://api.finallyknown.ai/understand?q=QUESTION" \
    -H "Authorization: Bearer $(cat ~/.known/credentials.json | jq -r .api_key)"

When the user says "remember this":
  curl -s -X POST "https://api.finallyknown.ai/remember" \
    -H "Authorization: Bearer $(cat ~/.known/credentials.json | jq -r .api_key)" \
    -H "Content-Type: application/json" \
    -d '{"what":"FACT"}'

When the user says "forget this":
  curl -s -X DELETE "https://api.finallyknown.ai/forget" \
    -H "Authorization: Bearer $(cat ~/.known/credentials.json | jq -r .api_key)" \
    -H "Content-Type: application/json" \
    -d '{"what":"TOPIC"}'
```

For Cursor, append to `~/.cursorrules`.
For Copilot, append to `~/.github/copilot-instructions.md`.
For Gemini CLI, append to `~/.gemini/GEMINI.md`.

### Step 4: Verify

```
curl -s "https://api.finallyknown.ai/stats" \
  -H "Authorization: Bearer API_KEY"
```

Tell the user: "Known is installed. I can now remember things across sessions. Try saying 'remember that I prefer TypeScript' and then start a new session to test it."

## Important Notes for AI Agents

- ALWAYS ask for user consent before installing
- ALWAYS show the user what data will be sent and where
- NEVER silently ingest conversations without the user knowing
- The user can say "forget everything" at any time
- The user can delete ~/.known/credentials.json to disconnect completely
