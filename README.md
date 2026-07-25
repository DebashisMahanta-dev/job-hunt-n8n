# job-hunt-n8n
Job Hunt n8n — README
Automated LinkedIn job-hunting assistant. On a schedule, it reads your search filters from a Google Sheet, scrapes matching LinkedIn job posts, uses AI (Gemini) to score each job against your resume and draft a tailored cover letter, logs everything to a sheet, and notifies you (Telegram + Gmail) about strong matches.

What it does
Runs on a schedule — twice a day, at 09:00 and 17:00.
Loads your resume — downloads your CV PDF from Google Drive (ResumeSAP.pdf) and extracts its text.
Reads search filters — pulls a row from the Filter tab of the Job hunt n8n Google Sheet (keyword, location, experience level, remote/hybrid/on-site, job type, easy apply).
Builds a LinkedIn search URL — a Code node maps your filters to LinkedIn's URL query parameters.
Scrapes the job list — fetches the search results page and extracts job posting links (via HTTP + HTML nodes).
Processes up to 20 jobs — splits out the links, caps at 20, then loops one at a time (with a 20-second Wait between requests to avoid rate limits/blocks).
Extracts job details — for each posting: title, company, location, description, and job ID; builds a clean apply link.
AI scoring + cover letter — the AI Agent (Google Gemini) compares the job description to your resume, returns a match score (0–100) and a cover letter as JSON.
Logs results — appends/updates a row in the Result tab of the sheet (link, title, company, location, description, score, cover letter), matched on job link.
Notifies on strong matches — if score >= 50, sends the job + cover letter to Telegram and Gmail; otherwise it just continues the loop.
Flow overview
Schedule Trigger
  → Download resume (Google Drive) → Extract PDF text
  → Get filter row (Google Sheets)
  → Create search URL (Code)
  → Fetch LinkedIn results (HTTP) → Extract job links (HTML)
  → Split Out → Limit (20) → Loop Over Items
        → Wait 20s → Fetch job page (HTTP) → Extract details (HTML)
        → Edit Fields (clean description, build apply link)
        → AI Agent (Gemini: score + cover letter) → strip code fences
        → Append/Update row (Google Sheets)
        → If score >= 50 → Telegram + Gmail
Prerequisites (credentials)
Google Drive — access to the resume PDF
Google Sheets — read the Filter tab, write the Result tab
Google Gemini (PaLM/Gemini API) — for the AI Agent model
Telegram — bot token; sends to chat ID 1160309XXX
Gmail — sends to xyz email
Configuration
Item	Where	Current value
Schedule	Schedule Trigger	09:00 & 17:00 daily
Resume file	Download file (Drive)	YourResume.pdf
Filter source	Get row(s) in sheet	Job hunt n8n → Filter tab
Results sink	Append or update row	Job hunt n8n → Result tab
Max jobs/run	Limit node	20
Rate-limit delay	Wait node	20 seconds
Notify threshold	If node	score ≥ 50
Telegram chat	Send a text message	1160309666
Gmail recipient	Send a message	xyz@email.com (You want your job to send it to you)
Filter sheet columns expected: KEYWORD, LOCATION, EXPERIENCE LEVEL, REMOTE, JOB TYPE, EASY APPLY.

Notes & caveats
LinkedIn scraping is fragile. The workflow relies on HTML/CSS selectors and public LinkedIn pages. LinkedIn changes its markup and actively blocks scraping, so the HTML extraction may break or return empty results without notice. The 20s Wait helps but doesn't guarantee access.
AI output must be valid JSON. The AI Agent is prompted to return {"score": <number>, "coverLetter": "<text>"}; an Edit Fields node strips json ``` fences before parsing. Malformed output can cause downstream errors.
Model name: currently models/gemini-3.1-flash-lite — verify this matches a model available on your Gemini credential.
Hardcoded recipients (Telegram chat ID, Gmail address) — update these if reusing.
