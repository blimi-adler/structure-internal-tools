# Team assessment JSON

This file is the contract for `team-assessment-tool.html`. A skill that prepares an import should write JSON in the shapes below. The page does not call Notion, Google, the Hub, or any other service. Someone opens the file in the browser, or an employee downloads a self-input file and the consultant opens that.

Harbor & Pine Remodeling, and the people on that sample, are fictional. Do not put a real client in an example or in this repo.

The page keeps work in `localStorage` under `structure.teamAssessment.v1`. Opening a file replaces or adds an assessment in that browser. It does not upload the file.

## Assessment file

Download **Download this assessment** writes this. **Open a saved file** reads it. A prior visit, used on the Actions step, is the same shape.

```json
{
  "app": "structure-team-assessment",
  "version": 2,
  "exportedAt": "2026-11-27T18:00:00.000Z",
  "assessment": {}
}
```

`version` 1 files still open. The page writes version 2. A bare assessment object, or `{ "assessments": [ ... ] }`, also opens. Any other `app` value is refused, except a performance review and a self-input file, which are imported into the open assessment instead of replacing it.

Fields the page keeps on `assessment`:

| Field | Type | Meaning |
| --- | --- | --- |
| `id` | string | Stable id. Re-opening the same id replaces that assessment. Omit it and the page mints one. |
| `sampleKey` | string | `"harbor-pine"` marks the built-in fictional sample. Leave this empty on a real file. |
| `companyName` | string | Client name. |
| `assessmentDate` | string | `YYYY-MM-DD`. |
| `consultantName` | string | Who from structure is in the room. |
| `createdAt`, `updatedAt` | string | ISO timestamps. Optional. |
| `view` | string | Last step. Optional. One of `home`, `setup`, `people`, `map`, `self`, `person`, `analysis`, `recs`, `future`, `actions`, `export`. |
| `activePersonId` | string | Person open on the Assess step. |
| `tradeTemplate` | string | Last starter list applied: `roofing`, `hvac`, `remodel`, `trades`, or `general`. Optional. |
| `reassessDate` | string | `YYYY-MM-DD`. The 30/60/90 check. |
| `reassessHorizon` | string | `"30"`, `"60"`, `"90"`, or `""`. |
| `people` | array | See below. |
| `activities` | array | See below. |
| `selfInputs` | array | Imported employee lists. See below. |
| `future` | object | `{ "seats": [], "moves": [] }`. See below. |
| `actions` | array | The working action list. See below. |
| `prior` | object or null | A previous assessment kept for the progress view. `{ "importedAt", "label", "assessment" }`. Do not nest `prior` inside that inner assessment. |

Unknown fields are dropped on import. Stick to this list.

### People

```json
{
  "id": "p_casey",
  "name": "Casey Nguyen",
  "role": "Scheduler and dispatcher",
  "department": "Operations",
  "manager": "Priya Shah",
  "pay": 51000,
  "startDate": "2023-02-14",
  "ratings": {
    "quality": 3,
    "reliability": 3,
    "communication": 4,
    "ownership": 3,
    "teamwork": 3
  },
  "strengths": "",
  "issues": "",
  "roleClarity": "partial",
  "fit": "workable",
  "notes": "",
  "recommendation": "coach",
  "reasoning": "",
  "roleAdjustments": "",
  "actionPlan": "",
  "performanceImport": null
}
```

- `pay` is a number of dollars a year, or `null`. It stays off the client owner report.
- `notes` are private to structure. They stay off the employee one-pager (when internal fields are hidden) and off the client owner report.
- `ratings` are integers 1–5 or `null`. 1 is below the role, 3 meets it, 5 is exceptional.
- `roleClarity` is `""`, `"yes"`, `"partial"`, or `"no"`.
- `fit` is `""`, `"strong"`, `"workable"`, or `"poor"`.
- `recommendation` is `""`, `"keep"`, `"coach"`, `"reassign"`, or `"replace"`. The client owner report does not print this call. It uses `roleAdjustments` and `actionPlan`.
- `performanceImport`, when present, records a ratings import. The page writes it. A generated file can omit it.

```json
{
  "importedAt": "2026-11-27T18:00:00.000Z",
  "reviewName": "Q4 2026",
  "matchedName": "Casey Nguyen",
  "gwc": { "g": "yes", "w": "no", "c": "yes" }
}
```

`gwc` values are `"yes"`, `"no"`, or `null`.

### Activities

The percent is a rough share of that person’s week, not a share of the activity.

```json
{
  "id": "act_sched",
  "name": "Scheduling",
  "department": "Operations/Field",
  "assignments": [
    { "personId": "p_casey", "role": "owner", "pct": 40 },
    { "personId": "p_chris", "role": "owner", "pct": 30 }
  ]
}
```

- `role` is `"owner"` or `"helper"`. Owner means accountable. More than one owner is an overlap. Zero owners is a gap. Helpers do not close a gap.
- `pct` is an integer 0–100.
- `department` should be one of: `Sales`, `Estimating`, `Operations/Field`, `PM`, `Purchasing`, `Finance`, `Admin`, `People/HR`, `Marketing`, `IT`, `Leadership`. An empty string is allowed. Any other string is kept and shown as its own group.
- `personId` must match a person `id`. Assignments that point at a missing person are dropped.

### Self-input stored on the assessment

This is the imported copy, not the file the employee downloaded. See the next section for that file.

```json
{
  "id": "self_casey",
  "personName": "Casey Nguyen",
  "role": "Scheduler and dispatcher",
  "companyName": "Harbor & Pine Remodeling",
  "exportedAt": "2026-11-17T15:00:00.000Z",
  "importedAt": "2026-11-18T12:00:00.000Z",
  "personId": "p_casey",
  "items": [
    { "name": "Scheduling", "department": "Operations/Field", "pct": 20 }
  ]
}
```

`personId` may be `""` when the name matches nobody yet. Matching is by the full name, ignoring case and punctuation. A unique first name also matches.

On screen, an activity name matches the map the same way. A difference of 15 points or more is a big gap. A line that exists on only one side is a missing activity.

### Future org

```json
{
  "seats": [
    { "id": "seat_coo", "title": "COO", "personId": "" },
    { "id": "seat_casey", "title": "Scheduler and dispatcher", "personId": "p_casey" }
  ],
  "moves": [
    { "activityId": "act_sched", "seatId": "seat_casey", "pct": 40 }
  ]
}
```

- A seat with an empty `personId` is an open seat.
- A person can fill only one seat.
- Each activity is on at most one seat. That move is the future owner. `pct` is that seat’s share of the week.
- Moves that point at a missing activity or seat are dropped.

### Actions

```json
{
  "id": "do_1",
  "key": "gap:act_purch",
  "source": "gap",
  "title": "Name an owner for Purchasing",
  "detail": "This activity has nobody accountable.",
  "clientTitle": "",
  "clientDetail": "",
  "owner": "Blimi Adler",
  "dueDate": "2026-12-18",
  "status": "open"
}
```

- `source` is `gap`, `overlap`, `overload`, `recommendation`, or `manual`.
- `status` is `open` (Not started), `doing` (In progress), or `done`.
- `key` is how **Build from this assessment** updates a line without wiping the owner, date, or status. Leave `key` empty on a line a person typed.
- For `recommendation` lines, `title` and `detail` are the working copy and can name the keep, coach, reassign, or replace call. `clientTitle` and `clientDetail` are what the owner report prints. Build those from the person’s role adjustments and next steps, not from the private call.

Keys the page generates:

| Key | Source |
| --- | --- |
| `gap:<activity id>` | No owner |
| `many:<activity id>` | More than one owner |
| `pair:<name>|<name>` | Two people share three or more activities. Names are normalized and sorted. |
| `side:<activity name>` | Owned only as a sliver of a larger seat |
| `over:<person id>` | Over 100% of a week |
| `thin:<person id>` | Five or more activities, none at 40% or more |
| `rec:<person id>` | A recommendation is set |
| `seat:<seat id>` | An open seat on the future org |

## Self-input file

The employee page writes this. The consultant opens it on **Self-input**, or with **Open a saved file** while an assessment is open. A second file for the same person replaces the first.

```json
{
  "app": "structure-team-assessment-self",
  "version": 1,
  "exportedAt": "2026-11-17T15:00:00.000Z",
  "companyName": "Harbor & Pine Remodeling",
  "personName": "Casey Nguyen",
  "role": "Scheduler and dispatcher",
  "items": [
    { "name": "Scheduling", "department": "Operations/Field", "pct": 20 },
    { "name": "Material runs", "department": "Purchasing", "pct": 15 }
  ]
}
```

`personName` should match the roster. `department` uses the same list as activities. `pct` is 0–100.

The mini page is a single HTML file the consultant downloads per person (`<name>-self-input.html`). It works offline. It only contains that person’s name, role, and company. It does not contain the rest of the assessment.

## Open seats (role list)

**Download open seats** writes this. It is the start of a job description: the seat title, and each activity with a percent of the week. Paste it into the role clarity tool or the hiring tool. This page only links to those tools. It does not send the file.

- Role clarity: `https://tools.ceomode.biz/role-clarity-tool.html`
- Hiring: `https://tools.ceomode.biz/hiring-tool.html`

```json
{
  "app": "structure-team-assessment-roles",
  "version": 1,
  "exportedAt": "2026-11-27T18:00:00.000Z",
  "companyName": "Harbor & Pine Remodeling",
  "seats": [
    {
      "title": "COO",
      "open": true,
      "loadPct": 40,
      "activities": [
        { "name": "Scheduling", "pct": 25 },
        { "name": "Hiring", "pct": 15 }
      ]
    }
  ]
}
```

The page does not import this file back. Generate an assessment, not a role list, when the skill is building a client file.

## Performance review import

Optional. On **Assess**, open a file downloaded from the performance evaluation tool (`https://tools.ceomode.biz/performance-evaluation-app.html`). People match by name. Pay in that file is ignored.

```json
{
  "app": "structure-performance-evaluation",
  "version": 1,
  "exportedAt": "2026-11-27T18:00:00.000Z",
  "session": {
    "name": "Q4 2026",
    "employees": [
      {
        "fields": { "empName": "Casey Nguyen" },
        "state": {
          "gwc": { "g": "yes", "w": "yes", "c": "no" },
          "evalState": {
            "cv_0": 2,
            "cv_1": 2,
            "cv_2": 1,
            "pf_0": 2,
            "pf_1": 2,
            "pf_3": 1
          }
        }
      }
    ]
  }
}
```

Also accepted: `{ "sessions": [ ... ] }`, or `{ "employees": [ ... ] }`.

Each question is `2` (consistently meets), `1` (inconsistently), or `0` (does not meet). The page translates that onto the 1–5 ratings:

| Review question | Rating |
| --- | --- |
| `pf_0` technical skill | `quality` |
| `pf_1` punctual and reliable, averaged with `cv_2` time management | `reliability` |
| `cv_1` communication | `communication` |
| `cv_0` ownership | `ownership` |
| `pf_3` working with others | `teamwork` |

`2` becomes 4, `1` becomes 2, `0` becomes 1. An average is rounded to the nearest integer. Empty scores are filled. Scores already on the assessment stay unless the consultant checks **Replace scores that are already filled in**. Get it / Want it / Capacity is stored on `performanceImport` and shown next to the person. It is not a 1–5 score.

## Action plan CSV

**Download CSV** on the Actions step writes a UTF-8 CSV with a header row:

```text
Name,Notes,Assignee,Due Date,Section,Status
```

- `Name` is the action.
- `Notes` is the working detail.
- `Assignee` is the owner line.
- `Due Date` is `YYYY-MM-DD`.
- `Section` is Gap, Overlap, Overload, Recommendation, Added, or Reassessment.
- `Status` is Not started, In progress, or Done.

If a reassessment date is set, the first data row is “Reassess the team” with that date. The working CSV can include the keep or replace wording. The printed client owner report does not.

## Progress view

On Actions, **Open a prior assessment** reads an assessment file and stores it on `prior`. The current assessment is not replaced. The comparison matches people and activities by name. It reports gaps that closed, gaps that remain, new gaps, overloads that resolved, recommendation changes, and load shifts of 10 points or more.

## What a skill should write

A full import is an assessment file with `people`, `activities` (including `department`), and optional `selfInputs`. Leave `future.seats` empty if the consultant will draw the next org in the page. Leave `actions` empty if they will press **Build from this assessment**.

Minimum that will open and show a map:

```json
{
  "app": "structure-team-assessment",
  "version": 2,
  "exportedAt": "2026-11-27T18:00:00.000Z",
  "assessment": {
    "companyName": "Harbor & Pine Remodeling",
    "assessmentDate": "2026-11-18",
    "consultantName": "Blimi Adler",
    "people": [
      { "id": "p_casey", "name": "Casey Nguyen", "role": "Scheduler", "department": "Operations" }
    ],
    "activities": [
      {
        "id": "act_sched",
        "name": "Scheduling",
        "department": "Operations/Field",
        "assignments": [{ "personId": "p_casey", "role": "owner", "pct": 40 }]
      }
    ]
  }
}
```
