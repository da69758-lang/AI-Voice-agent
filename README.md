Voice Agent with n8n

A browser-based AI voice assistant. You talk (or type), your words go to an n8n workflow powered by Google Gemini, and the answer is spoken back. Conversation history is saved per user in Supabase (Postgres), so it survives page refreshes and n8n restarts.

Voice agent screenshot

Features
Push-to-talk voice input using the browser's speech recognition
Spoken replies using the browser's text-to-speech
Text chat fallback
Per-user memory stored in Postgres (the browser keeps a permanent session ID)
Chat transcript restored after a page refresh
"Start new conversation" button to reset the session
Optional Header Auth token for the webhook
Single HTML file, no build step, no dependencies
How it works
Browser (index.html)
   |  speech to text (Web Speech API)
   v
POST /webhook/voice-agent   { message, sessionId, language }
   v
n8n workflow
   Webhook -> AI Agent -> Respond to Webhook
                |- Google Gemini Chat Model
                |- Postgres Chat Memory (Supabase)
   v
{ "reply": "..." }  ->  Browser speaks the reply (text to speech)
Requirements
n8n (self-hosted or n8n Cloud)
A Google Gemini API key
A free Supabase project (for the Postgres chat memory)
Chrome or Edge (speech recognition support)
Python 3, only to serve the page locally
Setup
1. Supabase
Create a project at supabase.com.
Choose a database password with letters and numbers only.
Open Connect -> Direct -> Session pooler and note the host, port, database, and user.
2. n8n credentials

Create these credentials in n8n:

Credential	Values
Google Gemini (PaLM) API	your Gemini API key
Postgres	Host: the Supabase pooler host, Port: 5432, Database: postgres, User: postgres.<project-ref>, Password: your DB password, SSL: Allow
Header Auth (optional)	Name: X-Auth-Token, Value: a long random token
3. n8n workflow

Build this workflow (or import workflow.json if included in this repo):

Webhook
Method: POST
Path: voice-agent
Respond: Using 'Respond to Webhook' Node
Option Allowed Origins (CORS): http://localhost:8000 (add your GitHub Pages origin later, comma separated, no path)
Authentication: Header Auth (optional)
AI Agent
Source for Prompt: Define below
Prompt (Expression): {{ $json.body.message }}
System Message: tell the agent to reply in 1 to 2 short spoken sentences with no markdown
Google Gemini Chat Model (connected to the agent's Chat Model input)
Use a Flash model and set a max token limit around 150 for faster replies
Postgres Chat Memory (connected to the agent's Memory input)
Session ID: Define below
Key (Expression): {{ $('Webhook').item.json.body.sessionId }}
Table name: n8n_chat_histories
Context window length: 10
Respond to Webhook
Respond With: Text
Response Body (Expression): {{ $json.output }}

Then click Publish. Edits only go live after you publish again.

4. Run the page locally
bash
python -m http.server 8000

Open http://localhost:8000/index.html, then in Settings paste your Production URL:

http://localhost:5678/webhook/voice-agent

Use the Production URL (/webhook/), not the Test URL (/webhook-test/).

If you enabled Header Auth, paste the same token into Auth header value.

Deploy the page on GitHub Pages
Push index.html to a public repository.
Go to Settings -> Pages, choose branch main and folder / (root), and save.
Your page will be at https://<your-username>.github.io/<repo-name>/.
Add https://<your-username>.github.io to the Webhook node's CORS option.

Important: GitHub Pages only hosts the page. Visitors' browsers call your n8n webhook directly, so localhost will not work for other people. For public use, n8n must be reachable over HTTPS, for example n8n Cloud, a VPS, or a tunnel such as Cloudflare Tunnel.

Security notes
Never commit API keys, database passwords, or tokens. The page does not hardcode any of them: the webhook URL and token are typed into Settings and stored only in the visitor's browser.
Set a spending limit on your Gemini API key.
A token typed into a public page is visible to whoever uses that browser, so Header Auth suits private use. For public demos, rely on spending limits and rate limiting instead.
Do not expose the n8n editor to the internet. Only the /webhook/ path needs to be reachable.
Use a CORS origin that matches your page exactly (origin only, no path).
Troubleshooting
Problem	Fix
"Could not reach n8n"	Check the URL uses /webhook/, the workflow is published, and CORS includes the page's origin
Empty reply from the webhook	Open Executions in n8n, find the red node, and read its error
"No session ID found" in the memory node	Set Session ID to Define below and use the Webhook expression above
Postgres connection fails	Use the Supabase Session pooler host, the user postgres.<project-ref>, and SSL Allow
Microphone does nothing	Use Chrome or Edge and open the page over localhost or HTTPS
Slow replies	Use a Gemini Flash model and limit the output tokens
Project structure
.
├── index.html      # the voice agent page
├── workflow.json   # exported n8n workflow (optional)
├── screenshot.png  # screenshot for this README (optional)
└── README.md

To export your workflow, open it in n8n, click the ... menu, and choose Download. Exports do not include credential secrets, but check the file before committing it.

Roadmap
Natural-sounding voices (ElevenLabs or OpenAI text-to-speech)
Tools for the agent: Google Sheets, Google Calendar, Gmail
Real phone calls through Vapi or Retell AI using the same n8n logic
License

MIT. Add a LICENSE file to your repository if you want this to apply.
