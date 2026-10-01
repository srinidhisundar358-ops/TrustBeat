# TrustBeat AI Backend

This backend creates one AI agent called **Ask TrustBeat**.

It combines:
- TrustBeat Buddy
- SafeTalk AI
- Diary AI Consultant

## Architecture

Browser / TrustBeat HTML
        |
        v
POST /ask-trustbeat
        |
        v
Intent detection
        |
        v
Privacy firewall
        |
        v
Only relevant user data
        |
        v
OpenAI Responses API
        |
        v
TrustBeat answer

## Why the privacy firewall matters

The frontend may have lots of user information, but the backend does NOT
automatically send all of it to the AI.

Examples:

"Why has my sleep score changed?"
-> sleep fields only

"Explain my blood report."
-> blood report fields only

"How can I manage exam stress?"
-> exam + diary fields only

"What patterns do you notice?"
-> wellness history fields used for pattern analysis

This is enforced in `agent.py` by `select_data()`.

## 1. Install

Use Python 3.12 or newer.

```bash
python -m venv .venv
```

Windows:

```bash
.venv\Scripts\activate
```

Then:

```bash
pip install -r requirements.txt
```

## 2. Configure the API key

Copy:

```text
.env.example
```

to:

```text
.env
```

Then put your API key in:

```text
OPENAI_API_KEY=your_real_key
```

Never put the API key inside your HTML or JavaScript.

## 3. Start the server

```bash
uvicorn main:app --reload
```

The API will run at:

```text
http://127.0.0.1:8000
```

Swagger testing page:

```text
http://127.0.0.1:8000/docs
```

## 4. Test request

POST to:

```text
/ask-trustbeat
```

Example:

```json
{
  "message": "Why has my sleep score changed?",
  "user_context": {
    "name": "Demo User",
    "sleep_score": 72,
    "sleep_history": [84, 81, 78, 72],
    "sleep_duration": 6.4,
    "steps_today": 9100,
    "allergies": ["peanuts"]
  }
}
```

Even though `steps_today` and `allergies` were supplied, the sleep question
will only expose the sleep fields to the AI.

## 5. Connect your TrustBeat HTML

Example JavaScript:

```javascript
async function askTrustBeat(message, userContext) {
    const response = await fetch("http://127.0.0.1:8000/ask-trustbeat", {
        method: "POST",
        headers: {
            "Content-Type": "application/json"
        },
        body: JSON.stringify({
            message: message,
            user_context: userContext
        })
    });

    if (!response.ok) {
        throw new Error("TrustBeat AI request failed");
    }

    return await response.json();
}
```

Then:

```javascript
const result = await askTrustBeat(
    "Why has my sleep score changed?",
    {
        sleep_score: 72,
        sleep_history: [84, 81, 78, 72],
        sleep_duration: 6.4
    }
);

console.log(result.answer);
```

## Suggested TrustBeat quick prompts

```text
Why has my sleep score changed?
What can I improve this week?
Explain my blood report.
How can I manage exam stress?
What patterns do you notice?
```

## Production upgrades

Before public deployment:
- authenticate every user
- identify users on the server rather than trusting a client-supplied user ID
- fetch user data from your MySQL database
- encrypt sensitive data at rest and in transit
- use HTTPS
- keep API keys server-side
- add proper authorization checks
- add stronger moderation/safety handling
- add audit logging without unnecessarily storing sensitive chat content
- validate uploaded blood reports before sending them to an AI model

The current version is intentionally beginner-friendly and designed to plug
into your existing TrustBeat HTML project.
