# Azure Devops MCP-Claude
AZURE DEVOPS MCP SERVER — SETUP GUIDE Connects Claude Desktop to your Azure DevOps account
================================================================
  AZURE DEVOPS MCP SERVER — SETUP GUIDE
  Connects Claude Desktop to your Azure DevOps account
================================================================


----------------------------------------------------------------
STEP 1 — INSTALL REQUIRED PROGRAMS
----------------------------------------------------------------

You need 3 programs installed before starting:

1. Node.js (v20 or higher)
   Download: https://nodejs.org
   Verify: open a terminal and run:  node --version

2. Docker Desktop
   Download: https://www.docker.com/products/docker-desktop
   Verify: open a terminal and run:  docker --version

3. Claude Desktop
   Download: https://claude.ai/download
   (Must be the desktop app, not the browser version)


----------------------------------------------------------------
STEP 2 — CREATE THE PROJECT FOLDER
----------------------------------------------------------------

Create this exact folder and file structure on your machine:

   ado-mcp/
   ├── src/
   │   └── index.ts
   ├── Dockerfile
   ├── package.json
   └── tsconfig.json

Important:
- The folder can be placed anywhere (e.g. C:\ado-mcp or ~/ado-mcp)
- File names must be exact — no extra words, no .txt extension
- index.ts goes INSIDE the src folder
- The other 3 files go in the root ado-mcp folder


----------------------------------------------------------------
STEP 3 — GET YOUR AZURE DEVOPS TOKEN (PAT)
----------------------------------------------------------------

1. Go to: https://dev.azure.com/YOUR-ORG-NAME
2. Click your profile picture (top right)
3. Click "Personal access tokens"
4. Click "+ New Token"
5. Fill in:
   - Name: claude-mcp (or anything you like)
   - Expiration: your choice
   - Scopes: select "Full access"
6. Click "Create"
7. COPY THE TOKEN — it is only shown once


----------------------------------------------------------------
STEP 4 — CREATE THE ATTACHMENTS FOLDER
----------------------------------------------------------------

Create a folder for file/screenshot attachments:

   Windows:   mkdir C:\attachments
   Mac/Linux: mkdir ~/attachments

This is where you place files before asking Claude to attach
them to a work item.


----------------------------------------------------------------
STEP 5 — BUILD THE DOCKER IMAGE
----------------------------------------------------------------

Open a terminal and navigate to your ado-mcp folder:

   Windows:   cd C:\ado-mcp
   Mac/Linux: cd ~/ado-mcp

Run these two commands in order:

   npm install

   docker build -t ado-mcp:latest .

Wait for both to finish. The second one takes 1-2 minutes
the first time. You should see "FINISHED" at the end.


----------------------------------------------------------------
STEP 6 — CONFIGURE CLAUDE DESKTOP
----------------------------------------------------------------

Open the Claude Desktop config file:

   Windows:   %APPDATA%\Claude\claude_desktop_config.json
   Mac:       ~/Library/Application Support/Claude/claude_desktop_config.json

   Tip (Windows): Press Win+R, paste %APPDATA%\Claude, press Enter

Add the following block inside the file. If the file already
has content, merge the "mcpServers" section into it — do not
replace the entire file:

   {
     "mcpServers": {
       "ado": {
         "command": "docker",
         "args": [
           "run", "--rm", "-i",
           "-e", "ADO_ORG_URL",
           "-e", "ADO_PAT",
           "-v", "C:\\attachments:/attachments",
           "ado-mcp:latest"
         ],
         "env": {
           "ADO_ORG_URL": "https://dev.azure.com/YOUR-ORG-NAME",
           "ADO_PAT": "PASTE-YOUR-TOKEN-HERE"
         }
       }
     }
   }

Replace:
   YOUR-ORG-NAME      → your Azure DevOps organization name
   PASTE-YOUR-TOKEN-HERE → the PAT token you copied in Step 3

On Mac/Linux, change the attachments path from:
   "C:\\attachments:/attachments"
to:
   "/Users/YOUR-USERNAME/attachments:/attachments"


----------------------------------------------------------------
STEP 7 — START EVERYTHING
----------------------------------------------------------------

1. Make sure Docker Desktop is open and running
   (look for the whale icon in the taskbar/menu bar)

2. Fully quit Claude Desktop and reopen it
   (right-click tray icon → Quit, then relaunch)

3. In Claude Desktop, click the + button in the message box
   then click Connectors — you should see "ado" listed
   with a blue toggle


----------------------------------------------------------------
STEP 8 — VERIFY IT WORKS
----------------------------------------------------------------

Start a new chat in Claude Desktop and say:

   "List all my Azure DevOps projects"

Claude should respond with your project list. If it does,
everything is working correctly.


----------------------------------------------------------------
THINGS TO KNOW
----------------------------------------------------------------

- Docker Desktop must be running whenever you use Claude
  with Azure DevOps tools. It does not need to show any
  active containers — just needs to be open.

- The "ado" server starts and stops automatically per request.
  You will not see a persistent container in Docker Desktop.

- PAT tokens expire. When yours expires, generate a new one
  (Step 3) and update the token in claude_desktop_config.json,
  then restart Claude Desktop.

- To attach a file or screenshot to a work item, place the
  file in C:\attachments first, then ask Claude to attach it.

- Project names, states, and board names do not need to be
  typed exactly. Claude will fuzzy-match what you say to the
  real names in Azure DevOps.


----------------------------------------------------------------
WHAT CLAUDE CAN DO WITH THIS MCP
----------------------------------------------------------------

Work Items:
  - Read, search, create, update any work item type
  - Post comments, attach files and screenshots
  - Add/remove tags, link work items, delete items

Test Plans:
  - List, create, update test plans and suites
  - Create test cases with steps
  - Add/remove test cases from suites
  - Create and manage test runs
  - Record pass/fail results on test cases
  - List test points and configurations

Boards & Project Info:
  - List projects, iterations, area paths
  - Get board columns and items

================================================================
