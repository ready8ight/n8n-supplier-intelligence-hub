# Setup Guide

Complete setup instructions for the N8N Supplier Vetting Automation workflow.

---

## Prerequisites

### 1. N8N Instance
- **Self-hosted**: Docker, npm, or cloud deployment
- **N8N Cloud**: [n8n.cloud](https://n8n.cloud) (managed service)
- **Version**: N8N 1.0+ recommended

### 2. Gmail Account
- Gmail account with OAuth2 access
- Gmail API enabled (N8N handles this during credential setup)
- Label created for tracking supplier emails

### 3. Notion Workspace
- Notion account (free or paid)
- Two databases created (see schemas below)
- Notion API integration created

### 4. FastAPI Backend (Optional)
- Python 3.9+ environment
- Anthropic API key (for Claude AI)
- Hosted endpoint (Railway, Render, Heroku, or self-hosted)

---

## Notion Database Setup

### Database 1: Suppliers (Master List)

**Purpose**: Central database of all suppliers with capacity, pricing, and quality data.

#### Required Fields:

| Field Name | Type | Description | Example |
|------------|------|-------------|---------|
| `Name` | Title | Supplier company name | "Kerala Coir Industries" |
| `Emails (All)` | Rich Text | All email addresses (for matching) | "info@keralacoir.com, sales@keralacoir.com" |
| `Capacity (MT/month)` | Number | Monthly production capacity | `100` |
| `Primary Port` | Select | Main export port | Options: Cochin, Chennai, Mumbai, Tuticorin, Colombo |
| `Payment Terms` | Rich Text | Payment conditions | "30% advance, 70% before shipping" |
| `Typical Lead Time` | Rich Text | Production + shipping time | "Production: 10 days, Shipping: 25 days" |
| `MOQ` | Rich Text | Minimum order quantity | "40 MT per 40ft container" |
| `Certifications` | Multi-select | Quality certifications | Options: EU Organic, OMRI, RHP, ISO 9001 |
| `Has Lab` | Checkbox | In-house lab testing | ✅ or ❌ |
| `Lab Tests` | Multi-select | Available lab tests | Options: EC, pH, Moisture, Fiber Length, Bulk Density |
| `Process Steps` | Multi-select | Manufacturing processes | Options: Washing, Screening, Composting, Buffering, Drying |
| `Key Machinery` | Rich Text | Main equipment | "3 washing drums, 2 buffering tanks, 5 drying yards" |
| `Primary Contact` | Rich Text | Main contact person | "Rajesh Kumar - Sales Manager" |
| `WhatsApp` | Phone | WhatsApp contact number | "+91 9876543210" |
| `Pricing Notes` | Rich Text | Historical quotes with dates | See format below |

#### Pricing Notes Format:
```
Latest Quote (2026-01-30):
• 5kg blocks: $180/MT FOB Cochin
• Capacity: 100 MT/month
• Production lead: 10 days
• Shipping lead: 25 days
• Payment: 30% advance
• Port: Cochin
• EC: <0.5 mS/cm
• pH: 5.5-6.5
• Moisture: <20%

Previous Quote (2026-01-15):
• 5kg blocks: $185/MT FOB Cochin
• Similar specs, price was higher before volume commitment
```

---

### Database 2: Supplier Communications (Email Log)

**Purpose**: Timestamped record of every supplier email with AI analysis.

#### Required Fields:

| Field Name | Type | Description | Example |
|------------|------|-------------|---------|
| `Subject` | Title | Email subject line | "Re: Quote for 5kg Blocks" |
| `Direction` | Select | Email direction | Options: Inbound, Outbound |
| `Status` | Select | Response status | Options: Replied, Pending, Follow-up Needed, Closed |
| `Sent Date` | Date | When email was sent | 2026-01-30 |
| `Last Reply Date` | Date | Last reply timestamp | 2026-01-30 14:35 |
| `Gmail Thread ID` | Rich Text | Gmail thread identifier (for tracking) | "18cf333d300f442681d5" |
| `Raw Email Body` | Rich Text | Full email text (truncated to 1900 chars) | "Hi, we can supply 100 MT/month at..." |
| `Supplier` | Relation | Link to Suppliers database | → Kerala Coir Industries |
| `Questions Answered MS` | Multi-select | Questions supplier answered | Options: capacity, pricing, certifications, lead_time, payment_terms, lab_testing |
| `Pending Questions MS` | Multi-select | Questions still unanswered | Options: (same as above) |
| `Extracted Data` | Rich Text | Full JSON from AI analysis | See format below |
| `Draft Response` | Rich Text | AI-generated follow-up email | "Hi Rajesh, Thank you for..." |
| `Draft Status` | Select | Draft review status | Options: Pending Review, Reviewed, Sent, Discarded |
| `Supplier Update Summary` | Rich Text | Audit trail of field updates | "Updated: Capacity, Primary Port, Certifications" |

#### Extracted Data JSON Format:
```json
{
  "capacity_mt_month": 100,
  "primary_port": "Cochin",
  "payment_terms": "30% advance, 70% before shipping",
  "production_lead_days": 10,
  "shipping_lead_days": 25,
  "ec_level_ms_cm": "<0.5",
  "ph_level": "5.5-6.5",
  "moisture_percent": "<20",
  "certs": ["EU Organic"],
  "contact_name": "Rajesh Kumar",
  "contact_whatsapp": "+91 9876543210",
  "process_qc": {
    "has_inhouse_lab": true,
    "lab_tests_available": ["EC", "pH", "Moisture"],
    "process_steps": ["Washing", "Buffering", "Drying"]
  },
  "pricing": {
    "line_items": [{
      "product_type": "5kg blocks",
      "price_per_unit": 180,
      "unit": "MT",
      "incoterm": "FOB",
      "port_from": "Cochin"
    }],
    "currency": "USD"
  },
  "questions_answered": ["capacity", "pricing", "certifications", "lead_time"],
  "questions_pending": ["payment_terms", "lab_testing"],
  "draft_response": "Hi Rajesh,\n\nThank you for the quick response..."
}
```

---

## N8N Workflow Setup

### Step 1: Import Workflow

1. Download `workflows/supplier-email-vetting.json` from this repo
2. Open N8N → **Workflows** tab
3. Click **Import from File**
4. Select the downloaded JSON file
5. Click **Save** (workflow is now in your N8N instance)

---

### Step 2: Configure Gmail Credentials

1. In N8N, go to **Credentials** → **+ Add Credential**
2. Search for **Gmail OAuth2**
3. Click **Create New**
4. Follow N8N's OAuth2 flow:
   - Click **Connect my account**
   - Sign in with your Gmail account
   - Grant permissions
   - N8N will handle the OAuth2 token exchange
5. **Save** the credential
6. Go back to the workflow → **Gmail Trigger** node
7. Select your new Gmail OAuth2 credential from the dropdown

---

### Step 3: Set Up Gmail Label

1. In Gmail, create a new label:
   - Click **More** in left sidebar
   - **Create new label**
   - Name it: **RFI-Suppliers** (or your preferred name)
   - Click **Create**

2. Copy the Label ID:
   - Go to Gmail settings → **Labels** tab
   - Find your label → Click **Show** (if hidden)
   - The Label ID is visible in the URL or settings

3. Update the workflow:
   - Open **Gmail Trigger** node in N8N
   - In **Filters** section → **Label IDs**
   - Paste your Label ID (e.g., `Label_9010414397365468569`)

---

### Step 4: Configure Notion Credentials

1. Create Notion Integration:
   - Go to [notion.so/my-integrations](https://www.notion.so/my-integrations)
   - Click **+ New integration**
   - Name: "N8N Supplier Automation"
   - Select your workspace
   - Click **Submit**
   - **Copy the Internal Integration Token** (starts with `secret_...`)

2. Share Databases with Integration:
   - Open your **Suppliers** database in Notion
   - Click **•••** (top right) → **Add connections**
   - Select your "N8N Supplier Automation" integration
   - Repeat for **Supplier Communications** database

3. Add Credential to N8N:
   - N8N → **Credentials** → **+ Add Credential**
   - Search for **Notion API**
   - Paste your Internal Integration Token
   - **Save**

4. Configure Database IDs in Workflow:
   - **Find Supplier in Notion** node:
     - Click **Database ID** dropdown
     - Select your **Suppliers** database (N8N will auto-populate)
   - **Create Supplier Communication** node:
     - Click **Database ID** dropdown
     - Select your **Supplier Communications** database

---

### Step 5: Configure FastAPI Backend (Optional)

**Skip this step if you don't have an AI backend yet** - the workflow will still log emails to Notion (just without AI extraction).

1. Update **Analyze Supplier Email** node:
   - Click the node to open settings
   - **URL** field: Replace with your FastAPI endpoint URL
     - Example: `https://your-api.railway.app/api/v1/supplier-email/analyze`
   - **Authentication**: Set if your API requires auth (Bearer token, API key, etc.)

2. Expected API Schema (for reference):
```json
// Request (POST)
{
  "supplier_id": "notion_page_id",
  "supplier_name": "Kerala Coir Industries",
  "subject": "Re: Quote for 5kg Blocks",
  "body": "Hi, we can supply 100 MT/month at...",
  "attachments": []
}

// Response (JSON)
{
  "capacity_mt_month": 100,
  "primary_port": "Cochin",
  "payment_terms": "30% advance, 70% before shipping",
  "questions_answered": ["capacity", "pricing"],
  "questions_pending": ["payment_terms", "lab_testing"],
  "draft_response": "Hi Rajesh, Thank you for..."
  // ... (see Extracted Data format above)
}
```

**Alternative**: Remove the **Analyze Supplier Email** node entirely. The workflow will skip AI analysis and just log raw emails to Notion.

---

### Step 6: Test the Workflow

1. **Activate the workflow**:
   - Toggle **Active** switch in top right of N8N editor

2. **Send a test email**:
   - Send yourself an email
   - Apply the **RFI-Suppliers** label in Gmail
   - Wait 1 minute (Gmail Trigger polls every minute)

3. **Check N8N execution log**:
   - N8N → **Executions** tab
   - Should see a successful execution
   - Click to view node-by-node results

4. **Verify Notion updates**:
   - Check **Supplier Communications** database → New row created
   - Check **Suppliers** database → Fields updated (if supplier existed and AI extracted data)

---

## Troubleshooting

### Gmail Trigger not firing
- **Check label ID**: Make sure it matches exactly in Gmail and N8N
- **Check OAuth2**: Re-authenticate if token expired
- **Check workflow is active**: Toggle should be ON (green)

### "Supplier not found" error
- **Check email matching**: The **Emails (All)** field in Suppliers database must contain the sender's email address
- **Format**: Use comma-separated list: `info@example.com, sales@example.com`

### Notion API errors (404, 400)
- **Check integration permissions**: Database must be shared with N8N integration
- **Check database IDs**: Notion API changes database IDs if you duplicate/move databases
- **Re-select databases**: Click the dropdown in each Notion node and re-select

### AI analysis errors
- **FastAPI endpoint down**: Check if your backend is running (test with curl/Postman)
- **API key issues**: Verify Anthropic API key is valid and has credits
- **Timeout**: Increase timeout in **Analyze Supplier Email** node settings (default 60s)

### Data not merging correctly
- **Check merge logic**: Review **Build Supplier Update (Merge Logic)** node code
- **Empty fields**: Merge logic skips empty values - this is intentional
- **Multi-selects not merging**: Check that field type in Notion is **Multi-select**, not Select

---

## Advanced Configuration

### Customize Merge Logic

Edit the **Build Supplier Update (Merge Logic)** node (Code/JavaScript) to change:
- Which fields get updated
- Merge rules (e.g., always update pricing, conservative for capacity)
- Pricing notes format
- Update summary message

### Add More Notion Fields

1. Add field to **Suppliers** database in Notion
2. Update **Update Supplier** node → **Properties** → Add property
3. Update merge logic code to extract/compare that field

### Change Gmail Poll Frequency

**Gmail Trigger** node → **Poll Times**:
- Default: Every minute
- Options: Every 5 min, 15 min, 30 min, hourly
- Lower frequency = fewer API calls (better for rate limits)

---

## Next Steps

Once the workflow is running:

1. **Monitor executions** for the first week
   - Check for errors
   - Verify merge logic behavior
   - Adjust AI prompt if needed

2. **Build Phase 1 workflow** (outreach batch sender)
   - Pull supplier emails from Notion
   - Send batch RFI/RFQ with template variables
   - Auto-assign Gmail labels

3. **Add enhancements**:
   - Slack notifications for high-priority replies
   - Attachment parsing (PDF quotes)
   - Multi-language support
   - Duplicate detection

---

## Support

Questions or issues? Open an issue on [GitHub](https://github.com/ready8ight/n8n-supplier-intelligence-hub/issues).

---

**Last Updated**: January 30, 2026
