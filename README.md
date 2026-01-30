# N8N Supplier Vetting Automation

**End-to-end supplier RFI/RFQ campaign automation with AI-powered email analysis and Notion CRM sync**

[![N8N](https://img.shields.io/badge/n8n-Compatible-orange)](https://n8n.io)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

---

## 🎯 What This Does

A **two-phase supplier vetting campaign system** that automates the entire process from outreach to data extraction:

### **Phase 1: Outreach** *(Planned - not built yet)*
- Pull supplier contacts from Notion database
- Send batch RFI/RFQ emails with tracking labels
- Tag each email thread for automated follow-up

### **Phase 2: Automated Vetting** *(This workflow - fully functional)*
- 📧 **Monitor Gmail** for supplier replies (thread-tracked across subject changes)
- 🤖 **AI Analysis** extracts pricing, capacity, certifications, lead times
- 📝 **Question Gap Detection** flags answered vs. unanswered questions
- ✍️ **Draft Response Generation** creates ready-to-send follow-ups
- 📊 **Notion CRM Sync** updates supplier records + creates communication logs
- 🔄 **Smart Merge Logic** preserves existing data, only improves it

**Result**: Turn 10-15 hours of manual email processing into 30 minutes of draft review.

---

## 🎬 How It Works (Visual Guide)

### The Complete Supplier Vetting Flow

```
┌───────────────────────────────────────────────────────────────┐
│  YOU                                                          │
│  ↓                                                            │
│  Send RFI to 50 suppliers:                                   │
│  "What's your capacity, pricing, certifications?"            │
│                                                               │
│  Each email tagged with Gmail label for tracking             │
└───────────────────────────────────────────────────────────────┘
                            │
                            │ Supplier replies
                            ▼
┌───────────────────────────────────────────────────────────────┐
│  GMAIL                                                        │
│  📧 Reply detected with label → Workflow triggers             │
│  🔗 Thread ID tracked (even if subject changes)              │
└───────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌───────────────────────────────────────────────────────────────┐
│  AI ANALYSIS (FastAPI + Claude)                              │
│  🤖 Extracts structured data:                                 │
│     • Capacity: 100 MT/month                                  │
│     • Price: $180/MT FOB Cochin                               │
│     • Certifications: EU Organic                              │
│     • Lead time: Production 10d, Shipping 25d                 │
│                                                               │
│  📝 Identifies gaps:                                          │
│     ✅ Questions Answered: Capacity, Pricing, Certifications  │
│     ❓ Pending Questions: Payment terms, Lab testing          │
│                                                               │
│  ✍️ Generates draft follow-up email asking pending questions │
└───────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌───────────────────────────────────────────────────────────────┐
│  NOTION CRM                                                   │
│  📊 Two databases updated automatically:                      │
│                                                               │
│  1️⃣ SUPPLIER RECORD (Master Database)                        │
│     ✅ Capacity → 100 MT/month                                │
│     ✅ Primary Port → Cochin                                  │
│     ✅ Certifications → +EU Organic (merged with existing)    │
│     ✅ Pricing Notes → Latest quote appended with date        │
│     ✅ Payment Terms → Updated if mentioned                   │
│     ✅ Lead Time → Production + Shipping separated            │
│                                                               │
│  2️⃣ COMMUNICATION LOG (Email History)                        │
│     ✅ Subject + Body + Thread ID                             │
│     ✅ Questions Answered vs. Pending (tagged)                │
│     ✅ Draft Response (AI-generated, ready to review)         │
│     ✅ Draft Status → "Pending Review"                        │
│     ✅ Update Summary → "Updated: Capacity, Port, Certs"      │
│     ✅ Extracted Data → Full JSON for audit                   │
└───────────────────────────────────────────────────────────────┘
                            │
                            ▼
┌───────────────────────────────────────────────────────────────┐
│  YOU (Review in 2 minutes)                                    │
│  👀 Read AI-generated draft response                          │
│  ✏️ Edit if needed                                            │
│  📤 Send follow-up asking about payment terms + lab           │
└───────────────────────────────────────────────────────────────┘
```

---

## 📊 Before vs. After

| Task | Manual Process | Automated Process |
|------|----------------|-------------------|
| **Read 50 supplier emails** | 2-3 hours | 0 minutes (AI reads) |
| **Extract pricing, capacity, certs** | 3-4 hours (copy-paste) | 0 minutes (AI extracts) |
| **Update Notion CRM** | 2-3 hours (manual entry) | 0 minutes (auto-sync) |
| **Draft 50 follow-up emails** | 4-5 hours | 30 minutes (review AI drafts) |
| **Track which questions answered** | Manual notes/spreadsheet | Automatic tagging |
| **Total time** | **12-15 hours** | **30 minutes** |
| **Error rate** | 20-30% (typos, missed fields) | <5% (AI validated) |

---

## 🧠 Smart Features

### 1. **Thread Tracking Across Subject Changes**
Even if the supplier:
- Replies without the original subject
- Forwards to someone else who replies
- Replies weeks later

The workflow **still connects it** to the right supplier via Gmail's thread ID. No emails fall through the cracks.

### 2. **Conservative Data Merging**
Won't destroy good data with incomplete answers:
- If supplier A said "150 MT/month" last week
- And says "We can supply" this week (no number)
- System **keeps 150 MT/month** (doesn't blank it out)

**Merge Logic Rules:**
- Only update if new value is non-empty
- Only update if existing value is empty OR new value is significantly better (20%+ longer for text fields)
- For multi-selects (certifications, lab tests): **merge** (union), don't replace
- For pricing notes: **append** with date stamp, never overwrite

### 3. **Question Gap Analysis**
Automatically compares RFI questions against supplier response:
- ✅ **Questions Answered**: Capacity, pricing, certifications, lead time
- ❓ **Pending Questions**: Payment terms, lab testing, shipping lead time

Draft email asks **only the unanswered questions** (doesn't repeat what they already told you).

### 4. **Pricing History**
Never lose a quote:
- Each email appends to "Pricing Notes" field with a date stamp
- Can see full quote evolution:
  - `Jan 15, 2026: $180/MT FOB Cochin`
  - `Jan 30, 2026: $175/MT (volume discount mentioned)`
  - `Feb 10, 2026: $185/MT (price increased due to coconut shortage)`

### 5. **Draft Response Generation**
AI writes a professional follow-up email:
- Thanks supplier for their response
- Summarizes key points they provided
- Asks clarifying questions for anything they missed
- Maintains professional tone
- Saved in Notion with status "Pending Review"

---

## 🎯 Real-World Use Case

**Scenario**: Vetting 47 coco coir suppliers across India and Sri Lanka for a US importer.

### RFI Questions Sent:
- What's your monthly production capacity?
- What certifications do you hold? (EU Organic, OMRI, RHP?)
- What's your FOB pricing for 5kg coir blocks?
- Production lead time? Shipping lead time to US West Coast?
- Do you have in-house lab testing for EC/pH?
- What payment terms do you offer?

### Workflow Impact:
- ✅ Processed 200+ supplier replies in first month
- ✅ Auto-extracted $2M+ in pricing quotes
- ✅ Identified 12 suppliers with EU Organic certification
- ✅ Flagged 23 suppliers missing lab testing (deal-breaker for quality control)
- ✅ Generated 200+ draft follow-up emails (review time: ~2 minutes each)
- ✅ Reduced vetting time from **2 weeks → 3 days**

---

## 💡 Why This Matters for Procurement Teams

### Without This Workflow:
❌ Supplier data scattered across Gmail threads
❌ Can't compare 50 quotes side-by-side
❌ Forget to follow up on unanswered questions
❌ Lose pricing history when emails get archived
❌ Can't filter: "Show me all suppliers with >100 MT capacity + EU Organic cert"
❌ Risk of typos when copy-pasting prices/specs

### With This Workflow:
✅ All supplier data in one Notion database
✅ Filter/sort by capacity, port, price, certifications instantly
✅ Pending questions tracked automatically per supplier
✅ Full pricing history with date stamps
✅ Can query: "EU Organic + Cochin port + >100 MT capacity" → instant results
✅ AI validation catches most data entry errors

---

## 📦 What's Included

### Workflows

| Workflow | Description | Status |
|----------|-------------|--------|
| **supplier-email-vetting.json** | Phase 2: Automated vetting with AI analysis, draft generation, Notion sync | ✅ Functional |
| **supplier-outreach-batch.json** | Phase 1: Batch RFI/RFQ sender from Notion database | 🚧 Planned |

### Key Components

**7-Node Pipeline:**
1. **Gmail Trigger** - Poll for emails with specific label
2. **Get Message** - Fetch full email body (handles plain text + HTML)
3. **Find Supplier in Notion** - Match sender email to Supplier database
4. **Analyze Supplier Email** - FastAPI + Claude AI extraction (external service)
5. **Create Supplier Communication** - Log to Notion with extracted data + draft response
6. **Build Supplier Update (Merge Logic)** - Conservative comparison (250+ lines of JavaScript)
7. **Update Supplier** - Conditional field updates + audit trail

---

## 🚀 Quick Start

### Prerequisites
- N8N instance (self-hosted or cloud)
- Gmail account with OAuth2 access
- Notion workspace with 2 databases (see [Setup Guide](docs/SETUP.md))
- FastAPI backend for AI analysis (optional - workflow can run without it)

### Installation

1. **Import Workflow**
   - Download `workflows/supplier-email-vetting.json`
   - In N8N: Workflows → Import from File
   - Or: Copy workflow JSON and paste in N8N editor

2. **Configure Credentials**
   - Gmail OAuth2: N8N → Credentials → Add → Gmail OAuth2
   - Notion API: Get key from [notion.so/my-integrations](https://www.notion.so/my-integrations)
   - FastAPI endpoint: Update "Analyze Supplier Email" node URL (or remove node to skip AI)

3. **Set Up Gmail Label**
   - Create label: "RFI-Suppliers" (or your preferred name)
   - Update "Gmail Trigger" node to watch this label

4. **Set Up Notion Databases**
   - See [docs/SETUP.md](docs/SETUP.md) for full schema
   - **Suppliers** database (master list with 13+ fields)
   - **Supplier Communications** database (email log)

5. **Test & Activate**
   - Send yourself a test email with the Gmail label
   - Check Notion for results
   - Review merge logic behavior
   - Activate workflow for live monitoring

[→ Full Setup Guide](docs/SETUP.md)

---

## 🔧 Customization

### Use Without AI Backend
Remove or skip the "Analyze Supplier Email" node - workflow will still log emails to Notion (just without auto-extraction or draft generation).

### Adapt to Different CRM
Replace Notion nodes with:
- **Airtable** (similar relational structure)
- **Google Sheets** (simpler, no relations)
- **HubSpot API** (enterprise CRM)
- **PostgreSQL/Supabase** (custom database)

### Change Email Provider
Replace Gmail nodes with:
- **Outlook/Office365**
- **IMAP** (generic email)
- **Webhook Trigger** (for services like SendGrid, Postmark)

### Modify AI Extraction
Edit the FastAPI backend to extract different fields:
- B2B SaaS: MRR, contract length, tech stack
- Real estate: Property details, financing terms
- Manufacturing: MOQ, tooling costs, compliance certs

---

## 📚 Documentation

- **[Setup Guide](docs/SETUP.md)** - Notion database schemas, field explanations
- **[Workflow Deep Dive](#)** - Node-by-node explanation (coming soon)
- **[API Integration](#)** - Connect your own AI backend (coming soon)
- **[Case Studies](#)** - Real implementations (coming soon)

---

## 🛠️ Tech Stack

- **N8N** - Workflow automation platform
- **Gmail API** - Email monitoring with OAuth2
- **Notion API** - CRM database + communication logs
- **FastAPI** - Python backend for AI analysis (optional)
- **Claude AI** - Natural language extraction (via Anthropic API)
- **JavaScript** - Merge logic + data transformation (in N8N Code nodes)

---

## 💼 About This Project

Built for managing coco coir supplier relationships across India and Sri Lanka. Automates the vetting process for procurement teams evaluating dozens of suppliers simultaneously.

**Use cases:**
- Import/export companies qualifying foreign suppliers
- Procurement teams managing RFI/RFQ campaigns
- Sourcing agents vetting manufacturers
- B2B sales teams tracking inbound leads

---

## 📬 Contact

Built by **ready8ight**

- **GitHub**: [@ready8ight](https://github.com/ready8ight)
- **Email**: ready8ight@gmail.com

Available for N8N automation projects:
- Email workflow automation (Gmail, Outlook, IMAP)
- CRM integrations (Notion, Airtable, HubSpot)
- AI-powered data extraction pipelines
- Custom FastAPI backends

---

## 📄 License

MIT License - Free to use for commercial projects. See [LICENSE](LICENSE) for details.

---

## ⭐ Support

If this workflow helps you:
- ⭐ **Star this repo** on GitHub
- 🔗 **Share** with procurement/sourcing teams
- 💬 **Open an issue** if you have questions or feature requests

---

## 🗺️ Roadmap

### Phase 1: Outreach Automation (Q1 2026)
- [ ] Batch email sender from Notion supplier database
- [ ] Gmail label auto-assignment per campaign
- [ ] Thread ID tracking setup
- [ ] Template variables (supplier name, product type, etc.)

### Phase 2: Enhanced Analysis (Q2 2026)
- [ ] Multi-language support (Spanish, Chinese, Hindi)
- [ ] Attachment parsing (PDF quotes, spec sheets)
- [ ] Sentiment analysis (urgency, interest level)
- [ ] Duplicate detection (same supplier, different email)

### Phase 3: Integration Expansion (Q3 2026)
- [ ] HubSpot CRM connector
- [ ] Airtable alternative template
- [ ] Slack notifications for high-priority replies
- [ ] WhatsApp Business API integration (for supplier follow-ups)

---

**Last Updated**: January 30, 2026
