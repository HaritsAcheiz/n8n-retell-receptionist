# AI Receptionist Template (n8n + Retell AI)

A production-ready template for building an automated voice receptionist using n8n for logic and Retell AI for natural voice interactions.

## 🚀 Features
- **Voice Interaction:** Natural conversation via Retell AI.
- **Lead Capture:** Automatically qualifies and saves leads to Airtable.
- **Appointment Booking:** Real-time scheduling via Cal.com.
- **Support Tickets:** Logs customer issues directly into Airtable.
- **Human Handoff:** Seamlessly transfer calls to human agents when needed.
- **Post-Call Automation:** Generates summaries, analyzes sentiment, and sends SMS follow-ups.

## 🛠️ Prerequisites
- **n8n:** Self-hosted or Cloud version.
- **Retell AI:** API Key and account.
- **Airtable:** Base for CRM and logs.
- **Cal.com:** For appointment scheduling.
- **OpenAI:** For summary and sentiment analysis.
- **Twilio:** For SMS follow-ups.

## 📦 Setup Guide

### 1. Environment Configuration
Copy the `.env.example` (or create a `.env` file) and fill in your credentials:
```env
# Retell AI
RETELL_API_KEY=your_key
WEBHOOK_VERIFICATION_TOKEN=your_token

# Airtable
AIRTABLE_PAT=your_pat
AIRTABLE_BASE_ID=your_base_id

# Cal.com
CALCOM_API_KEY=your_key
CALCOM_EVENT_TYPE_ID=your_id

# AI & Comm
OPENAI_API_KEY=your_key
TWILIO_ACCOUNT_SID=your_sid
TWILIO_AUTH_TOKEN=your_token
TWILIO_PHONE_NUMBER=your_number
```

### 2. Airtable Setup
Create an Airtable base with the following tables:
- **leads:** `lead_name`, `phone_number`, `email`, `service_requested`, `lead_status`, `notes`.
- **support_tickets:** `caller_name`, `caller_email`, `issue_description`, `status`.
- **tbl_call_logs:** `call_id`, `summary`, `sentiment`, `recording_url`, `duration_ms`.

## 🚀 Deployment (Hetzner / VPS)

To move from local development to production, follow these steps:

1.  **Prepare VPS:** Spin up an Ubuntu 22.04 instance (Hetzner CX21 is recommended).
2.  **Install Docker:** Run `curl -fsSL https://get.docker.com | sh`.
3.  **Clone Repo:** `git clone https://github.com/HaritsAcheiz/n8n-retell-receptionist.git`.
4.  **Configure Env:** Create `.env` from the prerequisites section.
5.  **Launch:** Run `docker compose up -d`.
6.  **Secure Access:** Set up a reverse proxy (like Nginx Proxy Manager or Caddy) to map your domain to port `5678` with SSL.

### 3. n8n Workflow Import (CLI Method)

Once your Docker containers are running, you can import the workflows via the n8n CLI. The `./shared` folder is already mapped to `/data/shared` inside your container.

Run these commands from your host terminal:

```bash
# Import all workflows from the shared directory
docker exec -it n8n n8n import:workflow --separate --input=/data/shared/
```

*Note: After importing, you will still need to manually create and link your API credentials in the n8n UI.*

### 4. Retell AI Agent Setup
1.  Create a new Agent in Retell AI.
2.  Paste the content of `shared/Retell_Agent_Prompt.md` into the **System Prompt**.
3.  Configure Tools in Retell:
    - `capture_lead`: Point to your n8n `/retell` webhook.
    - `book_appointment`: Point to your n8n `/retell` webhook.
    - `create_support_ticket`: Point to your n8n `/retell` webhook.
    - `transfer_call`: Point to your n8n `/retell` webhook.
4.  Set the **Call Ended Webhook** in Retell to point to your n8n `/retell-call-ended` endpoint.
5.  **Update Webhooks:** Update the tool URLs in Retell AI dashboard to point to your live domain (e.g., `https://n8n.yourdomain.com/webhook/retell`).

## 📄 License
MIT
