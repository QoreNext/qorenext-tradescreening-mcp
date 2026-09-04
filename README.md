# Qorenext Trade Screening MCP

> Sanctions and trade-screening due-diligence — via MCP

Qorenext Trade screening MCP is a [Model Context Protocol](https://modelcontextprotocol.io) server that enables Claude and other AI clients to run sanctions screening and trade-screening due-diligence directly in chat.

---
## Prerequisites:
•	Node.js and npm installed (npm is used to install the connector package).  
•	A QoreNext Trade Screening api key.  
•	An active product subscription is required to use Qore MCP.  
•	Administrator or standard access to edit files in your user profile.

## Get your API key

Sign up at **https://qorenext-app.azurewebsites.net/signup** to get your `QORENEXT_API_KEY`.

## Steps to get your API Key
1. Sign up at **https://qorenext-app.azurewebsites.net/signup**. 
2. Select Trade Screening. 
3. Buy your subscription plan. 
4. Open Profile → APIs. 
5. Click Generate API Key. 
6. Give your API key a name and click Create. 
7. Copy the API key for use in Claude. 
---

## How To Connect:

### A) Claude Code
```bash
claude mcp add --transport http qorenext-mcp \
  "https://mcp.qorenext.com/tradescreening" \
  --header "X-API-Key: YOUR_API_KEY"
```

### B) Claude Desktop

## Prerequisites:
•	Claude Desktop installed on Windows.  
•	Node.js and npm installed (npm is used to install the connector package).  
•	A QoreNext Trade Screening api key.  
•	An active product subscription is required to use Qore MCP.  
•	Administrator or standard access to edit files in your user profile.  

## Install Node.js  
- Download and install the **LTS version** from [nodejs.org](https://nodejs.org/).
- The installation includes **npm** and **npx**.
- Open **Terminal** (Mac) or **Command Prompt/PowerShell** (Windows).
- Verify the installation: `bash below commands  
node -v  
npx -v  

Both commands should display a version number.  
If you see "command not found", restart your terminal and try again.  

### ## Steps to Setup the Claude Desktop

### 1. Open Claude Desktop  
Launch the **Claude Desktop** application on your computer.  

### 2. Open Settings
Go to:
**Menu → File → Settings**

### 3. Open Developer Settings
In the Settings window, select **Developer** from the left-side menu.  

### 4. Open the Config File

Click **Edit Config** to open the `claude_desktop_config.json` file.  
Remove the existing command in the file and paste the configuration provided below in Json.

**Windows default location:**  
C:\Users\<YourUsername>\AppData\Roaming\Claude\claude_desktop_config.json or `~/Library/Application Support/Claude/claude_desktop_config.json` (Mac)

### 5. Add the QoreNext MCP Server
Add the following configuration to claude_desktop_config.json and replace qore_xxxxx with your real API key.

```json
{
  "mcpServers": {
    "qorenext-mcp-ts": {
      "command": "npx",
      "args": [
        "mcp-remote",
        "https://mcp.qorenext.com/tradescreening",
        "--header",
        "X-API-Key:qore_xxxxx"
      ]
    }
  }
}
```
Watch your JSON syntax
Make sure the JSON braces and commas match exactly — one missing comma will break the file.

### 6. Save the file
Save your changes (Ctrl+S) and close the editor.

Fully quit Claude Desktop by closing it, exiting it from the system tray/taskbar, and using Task Manager to End Task for any remaining Claude processes running in the background; then reopen Claude Desktop.

### C) Cursor / Windsurf
Add to `.cursor/mcp.json` or `.windsurf/mcp.json`:

```json
{{
  "mcpServers": {
    "qorenext-mcp-ts": {
      "command": "npx",
      "args": [
        "mcp-remote",
        "https://mcp.qorenext.com/tradescreening",
        "--header",
        "X-API-Key:qore_xxxxx"
      ]
    }
  }
}
```
```

---

## Tools

| Tool | Auth | Description |
|---|---|---|
| `health_check` | ❌ Public | Verify server is running, get version info |
| `submit_sanctions_screening` | ✅ API Key | Submit entities for sanctions screening against USA sanctions lists |
| `submit_trade_screening` | ✅ API Key | Submit a company for trade screening due-diligence |
| `get_trade_screening_status` | ✅ API Key | Poll status and retrieve the full due-diligence report |

---

### `health_check`
Verify the server is running. No API key required.
```
check if Qorenext Trade Screening MCP is running
```

---

### `submit_sanctions_screening`
Submit entities for sanctions screening against USA sanctions lists (OFAC, MEU,  EntityList, etc.).

**Required per entity:** `entityEnglishName` or `entityNativeName` (at least one)

**Optional:** `country`, `address`

```
Screen Acme trading LLC أكمي للتجار at 123 Azadi Street, Tehran for sanctions 

```

**Returns:** one result per submitted entity, each with `matchFound`, `bestMatchScore`, and `matches`.

**Example Response (Status 200):**
```json
{
  "success": true,
  "status_code": 200,
  "data": [
    {
      "entityEnglishName": "Acme Trading LLC",
      "entityNativeName": "أكمي للتجارة",
      "country": "Iran",
      "address": "123 Azadi Street, Tehran",
      "matchFound": false,
      "bestMatchScore": 0,
      "matches": []
    }
  ],
  "message": ""
}
```

---

### `submit_trade_screening`
Submit a company for trade screening due-diligence: denied/restricted-party list checks, red-flag keyword analysis, risk assessment, and negative-news findings.

**Required:** `address`, `website`, and `companyEnglishName` or `companyNativeName` (at least one)

**Optional:** `parentEntityEnglishName`, `parentEntityWebsite`

Qore's `/trade-screening/add-company` endpoint internally requires several
additional fields (`Labs`, `LabsWebsite`, `Department`, `DepartmentWebsite`,
`Professors`, `Product`, `EndUse`, `ParentEntity`, `referenceNumber`) beyond
what this tool asks for — the tool sends them as blank strings automatically,
callers never need to supply them.

**Example Input:**
```json
{
  "companyEnglishName": "Changchun New Industries Optoelectronics Tech. Co., Ltd.",
  "companyNativeName": "长春新产业光电技术有限公司",
  "address": "No.888 Jinhu Road High-tech Zone, Changchun 130103, P.R.China",
  "website": "https://test.com",
  "parentEntityEnglishName": "",
  "parentEntityWebsite": ""
}
```
`companyNativeName` may be omitted if `companyEnglishName` is provided (and vice versa). `parentEntityEnglishName`/`parentEntityWebsite` are optional.

```
run trade screening on Acme Corp at 1 Main St, https://acme.com
due-diligence check for Changchun New Industries Optoelectronics, https://cnilaser.com
```

**Returns:**
- `data`: Tracking id (`NativeId`) — use with `get_trade_screening_status`
- `message`: Description or error message

**Example Response (Status 200):**
```json
{
  "success": true,
  "status_code": 200,
  "data": 424,
  "message": ""
}
```

**Known issue (Qore backend, not this server):** new submissions currently
fail intermittently with `"Object reference not set to an instance of an
object."` (`status_code: 422`) — a server-side null-reference exception in
Qore's request-processing pipeline, not a client-side validation problem.
Duplicate submissions of already-known companies are unaffected. Confirmed
via direct testing against the live API as of 2026-08-19; pending a fix
from the Qore API team.

---

### `get_trade_screening_status`
Poll a trade screening request for status and the full due-diligence report.

**Parameter:** `request_id` (int) — the `NativeId` returned by `submit_trade_screening`

**Status values:** `Pending` · `Processing` · `Completed` · `Failed`

```
get status of trade screening request 424
check if screening 322 is complete
```

`output` (risk assessment, red-flag findings, restricted-party matches, recommendation) is only present once `status` is `Completed`.

---

## Example workflow

**Sanctions Screening:**
```
You:    Screen Acme Trading LLC in Iran for sanctions.

Claude: [calls submit_sanctions_screening]
        ✅ Screened. No match found (score: 0)
```

**Trade Screening:**
```
You:    Run trade screening on Acme Corp, 1 Main St, https://acme.com

Claude: [calls submit_trade_screening]
        ✅ Submitted. Tracking ID: 424

You:    Get status of trade screening request 424

Claude: [calls get_trade_screening_status]
        Status: Completed
        Risk Level: Low
        Recommendation: Proceed with standard due diligence.
```

---

## Support

- Issues: https://github.com/QoreNext/qorenext-tradescreening-mcp/issues
- Email: support@qorenext.com
- Docs: https://qorenext.com

---

## License

MIT
