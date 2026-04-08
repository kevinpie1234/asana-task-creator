# asana-task-creator

Railway cron service that creates an Asana task nightly. Designed to be duplicated
for each recurring task variation.

---

## Setup

### 1. Get an Asana Personal Access Token
- Go to https://app.asana.com/0/my-apps
- Click **Personal access tokens → Create new token**
- Copy it

### 2. Find your GIDs

Run these curl commands (replace `YOUR_TOKEN`):

**List your projects:**
```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" \
  "https://app.asana.com/api/1.0/projects?workspace=YOUR_WORKSPACE_GID&limit=50" \
  | python3 -m json.tool | grep -E '"gid"|"name"'
```

**List workspaces (to get workspace GID):**
```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" \
  "https://app.asana.com/api/1.0/workspaces" \
  | python3 -m json.tool
```

**List sections in a project:**
```bash
curl -s -H "Authorization: Bearer YOUR_TOKEN" \
  "https://app.asana.com/api/1.0/projects/YOUR_PROJECT_GID/sections" \
  | python3 -m json.tool | grep -E '"gid"|"name"'
```

**Find a template task GID:**
Open the task in Asana and grab the number from the URL:
`https://app.asana.com/0/{project_gid}/{task_gid}/f`

### 3. Edit `create_task.py`
Fill in the CONFIGURATION block at the top of the file:
- `TASK_NAME` — what the task will be called
- `ASSIGNEE` — employee's email address
- `PROJECT_GID` — from step 2
- `SECTION_GID` — from step 2 (or `None`)
- `DUE_DATE` — `"today"`, `"tomorrow"`, or `"YYYY-MM-DD"`
- `DESCRIPTION` — optional multi-line string, or `None`
- `TEMPLATE_TASK_GID` — optional, duplicates a template task instead of creating fresh

### 4. Deploy to Railway
```bash
# Create a new repo, push the files, then:
railway login
railway init      # create new project
railway up
```

Set the environment variable in Railway dashboard:
- `ASANA_TOKEN` = your personal access token

### 5. Set the cron schedule
In `railway.json`, the `cronSchedule` field uses UTC. 1am CT = 7am UTC:
- Standard time (Nov–Mar): `"0 7 * * *"`
- Daylight time (Mar–Nov): `"0 6 * * *"`

Or just pick one offset and accept a 1-hour seasonal drift.

---

## Duplicating for a new task

1. Copy this entire repo to a new folder / new GitHub repo
2. Edit only the **CONFIGURATION block** in `create_task.py`
3. Deploy as a new Railway service with the same `ASANA_TOKEN` env var
4. Done — each variation is its own isolated cron service

---

## Running locally (for testing)

```bash
pip install -r requirements.txt
export ASANA_TOKEN=your_token_here
python create_task.py
```
