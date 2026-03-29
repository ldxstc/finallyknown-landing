# Known — Installation Instructions (For AI Agents)

You are an AI agent helping your user install Known, a cross-session memory system. Follow these steps exactly.

## Step 1: Create a Known Account

Make an HTTP request:

```
POST https://api.finallyknown.ai/auth/auto-provision
Content-Type: application/json

{
  "agent_platform": "YOUR_PLATFORM",
  "device_id": "UNIQUE_DEVICE_HASH",
  "device_name": "USER_MACHINE_NAME"
}
```

For device_id, generate a hash of the machine's hostname. For agent_platform, use your name (e.g., "claude-code", "chatgpt", "cursor", "openclaw").

Save the response:
```json
{
  "api_key": "kn_live_xxxxxxxx",
  "user_id": "usr_xxx",
  "plan": "free"
}
```

Store the api_key in `~/.known/credentials.json`:
```json
{
  "api_key": "kn_live_xxxxxxxx",
  "api_url": "https://api.finallyknown.ai"
}
```

## Step 2: Configure Yourself

Add the following to your own configuration file so you query Known before every response:

### If you are Claude Code:
Append to `~/.claude/CLAUDE.md`:

```markdown
## Known Memory System
I have access to a persistent memory system at https://api.finallyknown.ai
My API key is stored in ~/.known/credentials.json

Before answering questions about the user, their preferences, projects, or history, I should query Known:
  curl -s "https://api.finallyknown.ai/understand?q=QUESTION" -H "Authorization: Bearer API_KEY"

After completing significant work, I should save a summary:
  curl -s -X POST "https://api.finallyknown.ai/ingest" -H "Authorization: Bearer API_KEY" -H "Content-Type: application/json" -d '{"text":"SUMMARY","source":"claude-code"}'

Available tools:
  GET /understand?q=...  — get context about the user
  GET /search?q=...      — search stored knowledge
  GET /grep?q=...        — search raw conversation history
  POST /remember         — store a specific fact
  DELETE /forget          — remove something from memory
  GET /stats             — check brain stats
```

### If you are Cursor:
Append to `.cursorrules` or `~/.cursorrules`:

```markdown
## Known Memory System
Query https://api.finallyknown.ai/understand before answering personal or project questions.
API key in ~/.known/credentials.json
```

### If you are OpenClaw:
Install the Known plugin: `clawhub install known`
Then restart: `openclaw gateway restart`

### If you are GitHub Copilot:
Append to `~/.github/copilot-instructions.md`:

```markdown
## Known Memory System
Query https://api.finallyknown.ai/understand for user context.
API key in ~/.known/credentials.json
```

### If you are Gemini CLI:
Append to `~/.gemini/GEMINI.md`:

```markdown
## Known Memory System
Query https://api.finallyknown.ai/understand for user context.
API key in ~/.known/credentials.json
```

### If you are ChatGPT (Custom GPT):
Add as an Action with this OpenAPI spec:
```yaml
openapi: 3.0.0
info:
  title: Known Memory API
  version: 1.0.0
servers:
  - url: https://api.finallyknown.ai
paths:
  /understand:
    get:
      operationId: getContext
      summary: Get user context
      parameters:
        - name: q
          in: query
          required: true
          schema:
            type: string
      security:
        - bearer: []
  /remember:
    post:
      operationId: rememberFact
      summary: Remember a fact
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                what:
                  type: string
      security:
        - bearer: []
components:
  securitySchemes:
    bearer:
      type: http
      scheme: bearer
```

## Step 3: Verify

Query Known to confirm it's working:
```
GET https://api.finallyknown.ai/stats
Authorization: Bearer YOUR_API_KEY
```

You should get back `{"nodes": 0, "insights": 0, ...}` — an empty brain, ready to learn.

## Step 4: Tell the User

Say: "Known is installed. I now have persistent memory across all your sessions. The more we talk, the better I'll understand you."

## Done

From now on:
- Query Known before answering personal/project questions
- Ingest important conversations after completing them
- Use /remember when the user says "remember this"
- Use /forget when the user says "forget this"
