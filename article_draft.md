How to Build a Production-Ready AI Receptionist Template (Retell AI + n8n)
Photo by Mikhail NilovIs it really possible to replace a human receptionist with AI?
Introduction
Right now, every client wants an AI receptionist, but building bespoke voice agents from scratch for every single business, wrestling with custom prompts, complex API webhooks, and endless conversational edge cases is a one-way ticket to ruined profit margins and developer burnout. The secret most automation agencies figure out the hard way is that 90% of front desk operations across all industries boil down to the exact same three tasks: booking appointments, routing support tickets, and capturing lead info.
Instead of reinventing the wheel every time a new dental clinic or roofing company signs a contract, you need a modular, production-ready template where the heavy lifting of API routing, error handling, and conversational guardrails is already done. By wiring up Retell AI as our conversational brain and n8n as our automated nervous system, we are going to build a master plug and play asset that lets you deploy a fully functional AI receptionist for a hair salon on Monday and a real estate brokerage on Tuesday just by swapping out a few simple variables.
Before we start connecting nodes and API keys, let's look at the blueprint of our 3-headed front desk automation. We aren't just making a robot that says "Hello", we are building a dynamic system that actively listens, makes decisions, and pushes data into your client's business tools in real-time. When a caller speaks, Retell AI processes their intent and triggers a specific "Custom Tool," sending a JSON webhook payload over to n8n. n8n then acts as the traffic cop, instantly firing off one of three workflows:
Scenario A (Booking) checks calendar and passes a success message back to Retell so the AI can verbally confirm the time
Scenario B (Support) pushes the issue into a ticketing system and returns a ticket ID for the AI to read aloud
Scenario C (Lead Capture) parses contact info straight into a CRM before the call even ends.

Ultimately, you are deploying one single AI agent equipped with three entirely different sets of invisible hands to do the heavy lifting in the background.
Pre-Requisites
To follow along and build this template, you don't need to be a senior software engineer, but you do need a few things set up:
A Retell AI Account: This is our voice engine. It handles the speech-to-text, the LLM thinking, and the text-to-speech.
Docker & Docker Compose: We are going to build our n8n instance locally first so we aren't burning server costs while tinkering. Docker keeps our local machine clean and makes deploying to a paid server later as easy as copy-pasting a file.
Ngrok: Since n8n will be running on your local computer, we need Ngrok to punch a secure, temporary tunnel through your Wi-Fi router so Retell AI's cloud servers can actually talk to it.
Hetzner VPS: Since this is a PoC, we are going to self-host n8n using a cheap, reliable Hetzner VPS. A basic instance (1–2 vCPUs, 2–4GB RAM) running Ubuntu with Docker and Docker Compose installed is perfect for running our automation engine without breaking the bank.
Cal.com Account: This is our developer-friendly scheduling engine.
Dummy CRM: To test the lead capture and support ticket features safely, set up a free Airtable base. It's fast, visual, and integrates beautifully with n8n.
Basic Webhook Knowledge: You should know what a POST request is and how JSON data looks. If you understand what { "name": "John" } means, you are overqualified.

That's it. No massive enterprise server deployments, no deep machine learning required. Just APIs talking to APIs.
Step by Step Tutorial
Step 1: Configuring the Retell AI Voice Engine
Before we build the nervous system (n8n), we need to build the brain and the mouth. Retell AI is a powerhouse for production-grade voice agents because it handles the hardest parts of conversational AI for you: speech-to-text, LLM routing, text-to-speech, and crucially interruption handling.
Head over to your Retell AI dashboard and hit Create New Agent.
Retell AI DashboardAI Agent Settings
When you create a new agent in Retell, you are immediately greeted by the Global Settings dashboard. This is where we build the core persona of your AI receptionist.
Voice & Language
A robotic voice ruins the illusion instantly. Based on your settings panel, here is how to configure the voice for maximum realism:
The Voice Selection: In the dropdown, choose a professional but warm voice like "Kathrine".
The Voice Model: Click the gear icon to open the advanced voice settings.
- Ensure Voice Model is set to Auto (Elevenlabs Turbo V2). ElevenLabs currently leads the industry in emotive, natural-sounding voice generation.
- Voice Speed: Leave this at 1.00. If you speed it up, the AI sounds nervous and rushed.
- Voice Temperature: This controls the emotional variance of the voice. Leaving it at 1.00 is perfect for a friendly receptionist. If you turn it down, the AI sounds more monotonous and strict, if you turn it up, it might sound overly dramatic.
- Voice Volume: Keep at 1.00 to match standard telephony levels.

Execution Mode
Retell offers two fundamental ways for the AI to handle a conversation. You must select Flex Mode. This gives the AI a shared context of all your tools and instructions, allowing it to act flexibly. If a user says, "I want to book an appointment… actually, wait, how much does a roof replacement cost?", Flex Mode allows the AI to smoothly abandon the calendar tool, answer the question, and pivot to the lead capture tool without crashing.
Global Prompt & LLM Selection
This is where you tell the AI exactly who it is and what it is allowed to do.
LLM: For a production-grade receptionist that needs to perfectly format JSON webhooks, you need reasoning model if available GPT 5.1 or Claude 4.6 Sonnet.
The Global Prompt: Do not treat this text box like a creative writing exercise. Do not give the AI a rich backstory. Treat it like a strict list of operating procedures. Paste this exact template into the box:

# ROLE
You are a highly efficient, professional front-desk receptionist for [COMPANY_NAME]. 
Your tone is warm, concise, and helpful. You never use filler words like "Umm" or "Ah."

# GOAL
Your goal is to assist the caller with ONE of three tasks:
1. Booking an appointment.
2. Logging a customer support issue.
3. Answering basic questions and capturing their contact info for a callback.

# INSTRUCTIONS
- If the user wants to book a time, ask for their preferred date and time, then immediately call the `book_appointment` tool.
- If the user has a problem or needs support, ask for a brief description of the issue, then immediately call the `create_support_ticket` tool.
- If the user just wants a quote or general info not in your knowledge base, ask for their name and phone number, then call the `capture_lead` tool.
- ALWAYS confirm success with the user AFTER a tool successfully runs.
Notice the [COMPANY_NAME] placeholder? That is what makes this a reusable template. When you deploy this for a new client, that's the only thing you change.
Agent Handbook
Retell's Agent Handbook injects pre-tested behavioral instructions directly into the LLM's brain behind the scenes.
Every toggle you flip ON adds hidden text to your system prompt. If you turn them all on, you bloat your context window by over 1,680 tokens. This dramatically slows down the AI's response time.
To get the optimal balance of speed, cost, and performance, use this mathematically lean setup:
Personality & Tone Tab:
- Professional Rep Personality: OFF (We already told the AI to be professional in our Global Prompt for a fraction of the token cost)
- Natural Filler Words: ON
- High Empathy: OFF
Accuracy & Format Tab:
- Echo Verification: ON
- NATO Phonetic Alphabet: OFF (Echo Verification is enough)
- Speech Normalization: OFF (Your base LLM is already smart enough to handle normal conversation without this massive overhead)
- Smart Matching (ASR / LLM Bridge): ON
Trust & Safety Tab:
- AI Disclosure When Asked: ON
- Scope Boundaries: ON

By using this specific Agent Handbook configuration,you get AI agent that gracefully masks API latency with natural filler words while n8n processes background tasks. Furthermore, strict Echo Verification and Smart Matching guarantee bulletproof data entry into your Cal.com and Airtable systems, while robust Scope Boundaries ensure the AI never hallucinates offers or deviates from its core instructions, resulting in a highly professional, reliable, and human-like caller experience without bloat the context window.
Knowledge Base
If the Agent Handbook is the AI's "behavioral training," the Knowledge Base is its memory. To demonstrate exactly how this works, we are going to use the dummy Apex Roofing & Solar FAQ.
Here is how to set it up for optimal retrieval accuracy:
1. Navigate to the Knowledge Base Tab In your Retell dashboard, leave the Agent settings and click on the Knowledge Base menu. Click "Create Knowledge Base" and name it Apex Roofing FAQ.
2. Upload the Dummy PDF You have three options for uploading data: URLs, Files, or Custom Text. For this build, we are going to use the Files option. Upload the Apex_Roofing_FAQ.pdf file you downloaded.
The "Q&A" Formatting Secret: If you open the PDF, you will notice it isn't just a giant block of marketing text. It is formatted with clean headers and specific Question/Answer pairs (e.g., "Q: How much does a commercial roof inspection cost? A: Commercial roof inspections start at $250."). LLMs parse Q&A formats incredibly well, guaranteeing the AI retrieves the exact right number instead of hallucinating.

3. Attach it to Your Agent Once your PDF finishes processing and the KB is saved, go back to your Agent's Global Settings. Scroll down to the Knowledge Base section, click the dropdown, and select the Apex Roofing FAQ you just built.
How the KB works with our n8n Webhooks:
Remember the instruction we put in our Global Prompt earlier?
- If the user just wants a quote or general info not in your knowledge base, ask for their name, phone number, and email, then call the capture_lead tool.
Because we set it up this way, here is exactly how the AI will handle a live call using the PDF we just uploaded:
Caller: "How much does a commercial roof inspection cost?"
AI: (Instantly searches the PDF) "Our commercial roof inspections start at $250. Would you like me to book a time for an inspector to come out?"
Caller: "What about installing a solar battery system?"
AI: (Searches the PDF, sees the strict policy on solar quotes) "We actually don't provide flat pricing for solar batteries - they are quoted on a custom basis depending on your energy needs. Could I get your name, phone number, and email so our sales team can reach out to you with an estimate?" (AI collects the info and fires the n8n webhook!)

By properly utilizing the Knowledge Base PDF, your agent sounds like a 10-year veteran of the company who knows every policy inside and out, while keeping your prompt tokens incredibly low and your latency blazing fast.
3. Setting Up Custom Tools
This is where the magic happens. A "Custom Tool" in Retell AI is essentially a bridge between the AI's brain and the real world. It uses a JSON schema to tell the LLM, "Hey, if the caller wants to achieve this specific goal, you must ask them for this exact information. Once you have it, send it to this URL."
In your Retell AI dashboard, navigate to the Tools or Custom Functions section. We are going to create three distinct tools.
For all three tools, configure the basic settings as follows:
HTTP Method: POST
Endpoint URL: Paste your Ngrok URL from Step 2, and append /webhook/retell to the end (e.g., https://a1b2-c3d4.ngrok-free.app/webhook/retell). Once you move to Hetzner, you will simply update this to your live server URL.

Here are the exact JSON schemas you need to create for each tool:
Tool 1: book_appointment
This tool handles scheduling via our Cal.com integration. 
{
  "type": "object",
  "properties": {
    "caller_name": {
      "type": "string",
      "description": "The first and last name of the caller."
    },
    "caller_email": {
      "type": "string",
      "description": "The email address of the caller to send the calendar invite to."
    },
    "requested_time": {
      "type": "string",
      "description": "The exact date and time the user wants to book, formatted as ISO 8601 (e.g., 2026-04-15T14:00:00Z). Calculate this based on the current date."
    }
  },
  "required": ["caller_name", "caller_email", "requested_time"]
}
Tool 2: create_support_ticket
If the caller has a complaint or a technical issue, the AI pivots from scheduling to support. This tool gathers the issue and drops it into your Airtable Support base.
{
  "type": "object",
  "properties": {
    "caller_name": {
      "type": "string",
      "description": "The first and last name of the caller."
    },
    "caller_email": {
      "type": "string",
      "description": "The email address of the caller so the support team can follow up."
    },
    "issue_description": {
      "type": "string",
      "description": "A detailed description of the customer's problem or complaint."
    }
  },
  "required": ["caller_name", "caller_email", "issue_description"]
}
Tool 3: capture_lead
This tool turns your AI into an inbound Sales Development Representative (SDR). If the caller asks for pricing, quotes, or general services, the AI collects their contact profile for the Airtable CRM.
{
  "type": "object",
  "properties": {
    "caller_name": {
      "type": "string",
      "description": "The first and last name of the caller."
    },
    "phone_number": {
      "type": "string",
      "description": "The best phone number to reach the caller."
    },
    "caller_email": {
      "type": "string",
      "description": "The caller's email address for sending quotes or information."
    },
    "intent": {
      "type": "string",
      "description": "A brief summary of what the caller is looking to buy, get a quote for, or learn about."
    }
  },
  "required": ["caller_name", "phone_number", "caller_email", "intent"]
}
What happens live on the phone?
The AI's LLM reads the "description" fields of these schemas to understand how to guide the conversation.
If a user says, "I'd like to book a meeting for next Tuesday at 2 PM," Retell's LLM instantly recognizes that this matches the book_appointment tool. However, because "caller_name" and "caller_email" are marked as required, the AI will not trigger the tool immediately. Instead, it will naturally ask: "I can certainly help you book that for next Tuesday at 2 PM. Could I please get your name and email address for the calendar invite?"
Once the AI collects all the required pieces, it quietly fills out the JSON form and fires the webhook straight into your n8n local tunnel. While n8n processes the database logic in the background, Retell automatically puts the caller on a brief, natural-sounding hold (saying something like, "Let me just pull up the calendar for you…") until n8n sends the success or error message back.
Step 2: Spinning Up a Production-Grade n8n Locally 
To keep this build radically efficient, we aren't going to burn cloud server costs while we are still tinkering with logic. Instead, we are going to build our n8n automation engine locally first.
But we aren't going to do a flimsy local install. We are going to use Docker Compose with a PostgreSQL database.
Why? Because you are building a perfectly portable, production-ready environment right on your laptop. The second your paid Hetzner server is active, you can instantly deploy this exact same setup with zero friction.
The Secrets (.env file)

Security first. We never hardcode passwords into our configuration files. Create a new folder on your computer (e.g., n8n-receptionist), open it in your code editor, and create a file named .env.
Paste this inside and change the passwords:
# N8N
GENERIC_TIMEZONE=Asia/Jakarta
N8N_ENCRYPTION_KEY=n8n-enscription-key-rahasia
N8N_USER_MANAGEMENT_JWT_SECRET=n8n-jwt-secret-rahasia
N8N_USER=n8nuserrahasia
N8N_PASSWORD=n8npasswordrahasia

# PostgreSQL
POSTGRES_USER=pguserrahasia
POSTGRES_PASSWORD=pgpasswordrahasia
POSTGRES_DB=n8n
The Engine (docker-compose.yml file)

In that same folder, create your docker-compose.yml file.
Paste this exact configuration inside:
services:
  n8n:
    container_name: n8n
    restart: unless-stopped
    image: n8nio/n8n:2.9.2
    networks: ['n8n_network']
    ports:
        - 5678:5678
    environment:
      - DB_TYPE=postgresdb
      - DB_POSTGRESDB_HOST=postgres
      - DB_POSTGRESDB_USER=${POSTGRES_USER}
      - DB_POSTGRESDB_PASSWORD=${POSTGRES_PASSWORD}

      # For Production
      # - N8N_BASE_URL=https://<your domain or sub-domain>/

      - N8N_PORT=5678
      - N8N_DIAGNOSTICS_ENABLED=false
      - N8N_PERSONALIZATION_ENABLED=false
      - N8N_ENCRYPTION_KEY
      - N8N_USER_MANAGEMENT_JWT_SECRET
      - GENERIC_TIMEZONE
      - N8N_COMMUNITY_PACKAGES_ALLOW_TOOL_USAGE=true
      - N8N_RUNNERS_ENABLED=true
      - N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true
    volumes:
      - n8n_data:/home/node/.n8n
      - ./shared:/data/shared
    env_file:
      - .env

  postgres:
    image: postgres:13.19-alpine
    hostname: postgres
    networks: ['n8n_network']
    restart: unless-stopped
    environment:
      - POSTGRES_USER
      - POSTGRES_PASSWORD
      - POSTGRES_DB
    ports:
      - 5432:5432
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -h localhost -U ${POSTGRES_USER} -d ${POSTGRES_DB}']
      interval: 5s
      timeout: 5s
      retries: 10
    env_file:
      - .env

volumes:
  n8n_data:
  postgres_data:

networks:
  n8n_network:
Booting Up the Stack

Open your terminal, navigate to your folder, and run the magic command: docker compose up -dDocker will pull the images, set up the secure network, initialize Postgres, and boot up n8n. Give it about 30 seconds, then open your browser to http://localhost:5678. Create your admin account, and you'll see your fresh n8n canvas.
The Tunnel Hack (Connecting Retell to Localhost)

Here is the final piece of the local puzzle. Retell AI lives on the public internet. It cannot send a webhook to your private localhost:5678. We need to punch a temporary, secure hole through your network so Retell can reach n8n. The easiest tool for this is Ngrok. Open a new terminal window and run this command:
ngrok http 5678
Ngrok will instantly generate a public URL. Copy that URL. Head back to your Retell AI Dashboard > Custom Tools. Paste that ngrok URL into the Endpoint URL field for your tools, and append /webhook/retell to the end of it (e.g., https://a1b2-c3d4.ngrok-free.app/webhook/retell).
Every time Retell AI decides to use a Custom Tool, it will now beam the payload from the cloud, straight through the Ngrok tunnel, and directly into the n8n container running on your laptop.
Step 3: Building the n8n "Traffic Cop" (Webhook & Router)
Right now, Retell AI is ready to send data, but n8n isn't listening. We need to build the entry point for our automation - a universal receiver that catches the payload, figures out what the user wants, and routes it to the right department.
Open your browser, go to http://localhost:5678, and create a brand new workflow. Name it something professional (or just name it "Robo-Brenda", I won't judge).
The Front Door (The Webhook Node)

Double-click on the canvas and add a Webhook node. This is the front door to your entire operation.
Configure it with these exact settings:
HTTP Method: POST
Path: retell (This matches the /webhook/retell URL we put into Retell's dashboard in Step 2).
Respond: Change this from the default to "Using 'Respond to Webhook' Node".
Webhook node settingThe Traffic Cop (The Switch Node)

When Retell hits this webhook, it sends over a JSON payload containing the name of the tool it wants to use and the arguments the caller provided.
It will look something like this:
{
  "name": "book_appointment",
  "args": {
    "caller_name": "John Doe",
    "requested_time": "2026-03-15T14:00:00Z"
  }
}
We don't want to build three separate webhooks for our three different scenarios. We want one smart webhook that routes the traffic.
Add a Switch node and connect it to your Webhook node. We are going to configure the Switch to look at the tool name coming from Retell. Set the Value 1 to an expression: {{ $json.body.name }} (This targets the tool name in the webhook payload). Create three routing rules:
Rule 1: If Value equals book_appointment -> routes to Output 0
Rule 2: If Value equals create_support_ticket -> routes to Output 1
Rule 3: If Value equals capture_lead -> routes to Output 2
Switch node setting Now, you have a beautiful, organized fork in the road. One single AI agent can trigger three completely different business processes.
Talking Back to Retell (The Respond to Webhook Node)

Before we build the actual calendar and CRM integrations, you need to understand how the conversation loop closes.
At the very end of each of the three paths we are about to build, you will place a Respond to Webhook node. This node sends data back to Retell AI so the agent knows what to say next.
n8n webhook workflow ended with Response to Webhook NodeIf the booking is successful, your Respond to Webhook node will pass back a simple JSON object:
{
  "status": "success",
  "message": "Appointment successfully confirmed for 2:00 PM.",
  "booking_reference": "CAL-9876"
}
When Retell receives this response, the LLM reads it, processes the success, and translates it into natural human speech for the caller: "Alright John, you are all set for 2 PM. Your reference number is CAL-9876. Is there anything else I can help you with today?"
Step 4: Building the 3 Core Action Workflows (The Hands)
Right now, your n8n workflow looks like a fork in the road with three empty paths. We are going to build the logic for each one. The structure for every path is always the same: Action Node (to do the work), Set Node (to format the result) and Respond to Webhook (to tell the AI).
Branch 1: The Appointment Booking
When the Switch node detects book_appointment, the payload goes down Path 0. Here is what you put there:
The Action Node (HTTP Request): Add an HTTP Request node to talk directly to the Cal.com v2 API.
- Method: POST 
- URL: https://api.cal.com/v2/bookings
- Authentication: Under credentials, set up a Header Auth. Add Authorization as the Name and Bearer YOUR_CAL_API_KEY as the Value.
- Headers: You must add a custom header for the v2 API to work.
Name: cal-api-version
Value: 2026-02-25 
- On Error Setting: In setting navigation tab, choose "Continue (using error output)
- Body Parameters: Send a JSON body containing the exact structure Cal.com requires. Map the variables Retell sent you into the attendee object:

{
  "start": "{{ $json.args.requested_time }}",
  "attendee": {
    "name": "{{ $json.args.caller_name }}",
    "email": "{{ $json.args.caller_emai }}",
    "timeZone": "America/New_York",
    "language": "en"
  },
  "eventTypeId": 5250255,
  "allowConflicts": true
}
- Settings
Cal.com http post request create booking parameters settingThe Set Node, Two Payload Formatters (Edit Fields Nodes)

Connect to the True output (The Success Path):
- Add another Edit Fields node
- Paste this exact payload:
{
  "status": "success",
  "message": "Appointment booked successfully.",
  "booking_uid": "{{ $json.data.uid }}"
}
Connect to the Error Branch output (The Error Path):
- Add an Edit Fields node.
- Paste this exact payload:
{
  "status": "error",
  "message": "The requested time slot is unavailable. Please apologize and ask the user for a different time or day."
}
The Respond to Webhook Node

Now, simply click and drag the output wire from your "Success" Edit Fields node into the Respond node. Then, click and drag the output wire from your "Error" Edit Fields node into the exact same Respond node.
Complete flow for appointment booking Set it to return the JSON you just formatted. The AI will read this and say, "You are booked! I'll send the confirmation."
Branch 2: The Support Ticket
Airtable is the absolute best tool for this PoC. It gives you a highly visual, instant backend without needing to write SQL or deploy a heavy CRM. Plus, your clients will love looking at it.
Database Preparation

Before n8n can log a ticket, we need a place to put it.
Create the Base and Table
- Log into Airtable and create a new Base. Name it something like "ai_receptionist_crm".
- Rename the default table to support_tickets.
Configure the Columns (Fields) Delete the default columns Airtable gives you and create exactly these four columns:
- ticket_number (Autonumber).
- issue_description (Long text).
- caller_name (Single line text).
- caller_email (Email).
- created_at (Created time) Use ISO for date format, check the include time check box and use 24 hours as time format.
- last_modified (Last modified time) Use ISO for date format, check the include time check box and use 24 hours as time format. Airtable will automatically update this timestamp every time someone on your support team changes the ticket status from "open" to "resolved".
- status (Single select) Add three options: open, in progress, and resolved. Set the default value to open.

3. Generate Your Personal Access Token. You must use a Personal Access Token (PAT) for n8n to connect.
- In Airtable, click your profile icon in the top right and go to Builder Hub.
- Click Personal access token in side navbar.
- Click Create new token.
- Name it "n8n_ai_integration".
- Under Scopes, add data.records:read, data.records:write and schema.bases:read.
- Under Access, select the specific Base you just created.
- Click Create Token. Copy this long string immediately, as Airtable will never show it to you again.
airtable database for support_tickets recordBuilding the n8n Logic

Now that your database is waiting, let's go back into your n8n canvas. When the Switch node detects create_support_ticket, the payload goes down this branch.
1. The Action Node
- Connect an Airtable node to the create_support_ticket output of your Switch node.
- Under Credentials, create a new Airtable Personal Access Token API credential and paste your token there.
- Configure the node:
Operation: Create Record
Base: Select "ai_receptionist_crm" from the dropdown.
Table: Select "support_tickets" from the dropdown.
Under Columns To Match/Send, click Add Field.
1. Add caller_name and map the value to:
 {{ $json.args.caller_name }} 
2. Add caller_email and map the value to:
 {{ $json.args.caller_email }}
3.Add issue_description and map the value to:
 {{ $json.body.args.issue_description }}
(Note: You do not need to map the ticket_number or the status. Airtable will automatically generate the Autonumber and apply the default "open" status the second the row is created!)
2. The Formatting Node (Edit Fields) When the Airtable node succeeds, it outputs all the data from the newly created row. We need to grab that human-readable ticket_number and format a nice response for the AI.
- Connect an Edit Fields node right after the Airtable node.
- Paste this exact payload:
{
 "status": "success",
 "ticket_number": {{ $json.fields.ticket_number }},
 "message": Support ticket logged successfully.
}
3. The Respond Node
Click and drag the output wire from your Edit Fields node directly into your Respond to Webhook node.
What happens live on the phone? The caller says, "My internet has been down for three hours!" Retell sends the payload. n8n drops it into Airtable. Airtable assigns it "Ticket 14". n8n replies to Retell. The AI instantly says: "I have logged that issue for you. Your reference number is Ticket 14, and our support team will be looking into it immediately."
Branch 3: The Lead Capture
This is where your AI transforms from a simple receptionist into a 24/7 inbound Sales Development Representative (SDR). If a caller asks about pricing, services, or wants a quote, the AI will gather their contact info and drop it straight into your sales pipeline.
Since we already set up the Airtable Base and the Personal Access Token in Branch 2, this branch is going to be incredibly fast to build.
Database Preparation

We are going to keep everything organized in the exact same Airtable Base, just on a new tab.
1. Create the Table
- Open your "ai_receptionist_crm" base in Airtable.
- Click the + icon next to your "support_tickets" tab to create a new empty table.
- Name this new table "leads".
2. Configure the Columns
Delete the default columns and create these specific fields for your sales team:
- lead_name (Single line text)
- phone_number (Phone number)
- email (Email)
- service_requested (Long text) This is where the AI will summarize what the caller actually wants.
- lead_status (Single select) with options: new_lead, contacted, disqualified, closed_won. Set the default to new_lead.
- created_at (Created time) Use ISO for date format, check the include time check box and use 24 hours as time format.
- last_modified (Last modified time) Use ISO for date format, check the include time check box and use 24 hours as time format. Airtable will automatically update this timestamp every time someone on your sales team changes the status.
airtable database for leads recordBuilding the n8n Logic

We are going to build the final branch coming out of your Switch node (capture_lead).
1. The Action Node (Airtable)
- Connect a new Airtable node to the capture_lead output of your Switch node.
- Select your existing Airtable credential.
- Configure the node:
Operation: Create Record
Base: Select "ai_receptionist_crm".
Table: Select "leads".
Under Columns To Match/Send, map the four fields from Retell:
Lead Name: {{ $json.body.args.caller_name }}
Phone Number: {{ $json.body.args.phone_number }}
Email: {{ $json.body.args.caller_email }}
Service Requested: {{ $json.body.args.intent }}

2. The Formatting Node (Edit Fields) We don't need to return a ticket number this time. We just need to tell the AI that the sales team got the message so it can wrap up the phone call gracefully.
Connect an Edit Fields node right after the Airtable node.
Create two String assignments:

Name: status | Value: success
Name: message | Value: Lead captured successfully. The sales team has been notified.

3. The Respond Node
Just like the other branches, simply click and drag the output wire from this new Edit Fields node directly into your single, universal Respond to Webhook node at the end of your canvas.
What happens live on the phone? Caller: "I'm looking to get my roof replaced next month and wanted to get a quote." AI: "I can absolutely help you get a quote for a roof replacement. What is the best name, phone number, and email to reach you at?" (Caller provides info. Retell fires the webhook. n8n drops it into Airtable and replies). AI: "Perfect. I've got all your details down and I just pinged our sales team. Someone will call you back shortly with that quote. Have a great day!"