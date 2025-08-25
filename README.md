# Alation Documenet Cloner (Inline Button) - README

## Overview
This is a pure-HTML button you paste into an Alation document’s Description field to clone that document into a target folder (e.g., “Template Output”) in the same collection (doc hub).
- No `<script>` tags (works in sanitized rich text).
- Uses your logged-in session + CSRF (no API token needed).
- Asynchronous create via Integration API, then polls the web-app list until the new doc appears.
- Adds an invisible GUID marker for detection and then removes it from the clone.
- Strips the cloner block from the source Description so it doesn’t copy into the clone.
- Adds a timestamp to the new title: Original Title (MM-DD-YYYY HH:MM) in local time.

## What it does (at a glance)
1. Collects CSRF from /account/auth/.
2. Reads the source doc via /integration/v2/document/?id=<current_id>.
3. Finds the collection & target folder:
   - Collection (aka doc hub) via /api/v1/documentation/ancestors/glossary_term/<current_id>/
   - Target folder “Template Output” via /api/v3/glossary/?collection_type_id=<hub_id>…
   - Falls back to the current folder if “Template Output” isn’t found.
4. Snapshots the collection list via /api/v2/term/?collection_type_id=<hub_id>… for polling.
5. Builds the clone payload, cleaning the Description:
   - Removes the cloner UI (wrapper + fallbacks).
   - Appends a hidden GUID marker to Description for detection.
   - Adds a timestamp to the title.
6. Creates the document via POST /integration/v2/document/ (array body) with CSRF.
7. Polls the collection list until a new item containing the GUID appears.
8. Cleans up the cloned doc by removing the GUID via PUT /integration/v2/document/.
9. Shows a Status line and an “Open new document” link.

## Installation
1. Create a new Document Hub for templates.
2. Create 1 or more folders to hold the templates.
3. Create a folder called "Template Output".
4. Create or open your template document (the one users will clone from).
5. Edit the Description field.
6. Switch the HTML/code view.
7. Paste the full cloner block (clone_document.html) at the bottom of Description.
8. Save the document.
9. Test by clicking "Clone This Template."

## Endpoints Used:

| Purpose                            | Endpoint                                                         | Method            | Auth                                                 |
| ---------------------------------- | ---------------------------------------------------------------- | ----------------- | ---------------------------------------------------- |
| Get CSRF                           | `/account/auth/`                                                 | GET               | Session (cookies), returns CSRF in page              |
| Read source doc                    | `/integration/v2/document/?id=<id>&deleted=false&limit=1&skip=0` | GET               | Session+CSRF not required for GET                    |
| Get collection info                | `/api/v1/documentation/ancestors/glossary_term/<id>/`            | GET               | Session                                              |
| List folders in collection         | `/api/v3/glossary/?collection_type_id=<hub_id>…`                 | GET               | Session                                              |
| List terms in collection (polling) | `/api/v2/term/?collection_type_id=<hub_id>…`                     | GET               | Session                                              |
| Create clone                       | `/integration/v2/document/`                                      | POST (array body) | **Session + CSRF**                                   |
| Cleanup marker                     | `/integration/v2/document/`                                      | PUT (array body)  | **Session + CSRF** (may be token-gated in some envs) |