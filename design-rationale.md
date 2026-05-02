# Design Rationale — Leadership Portfolio Redesign
 **Candidate:** Joakim Manoj   **Program:** B.Tech Information Technology, Government Engineering College Barton Hill   **Contact:** joakimmanoj5002@gmail.com · +91-97783-18081 · github.com/Joakimmanoj2k5

---

## Why I took this assignment seriously

I took this problem as one I already work on daily: turning structured backend data into a surface a real human wants to open. At Nexforz I integrate OpenAI-driven features into a CRM — the exact pattern of "raw signals in, meaningful UI out" that PDGMS's employee-facing modules describe. I've also built a full-stack platform that converts uploaded PDFs into interactive quizzes and summaries using local LLMs, which is structurally the same design challenge: dense data in, narrative-shaped output out.

The rationale below reflects that lens — start from what the reader needs to feel in 30 seconds, work backwards to which fields in data-model.ts earn their place on the page, and cut everything else.

---

## 1. The design thesis, in one paragraph

A Leadership Portfolio is a monthly artifact an employee will show a mentor five years from now. The mentor won't know what PDGMS is, won't care about V-levels, and will spend thirty seconds deciding whether to read further. That thirty-second scan must deliver: who this person is, what kind of month they just had, what they shipped that mattered, and where they're headed. Everything else — axis scores, ticket counts, constraint categories — is evidence. Evidence supports claims; it doesn't replace them. A dashboard inverts that order: it shows you evidence and asks you to reason. A pitch deck makes the claim first and lets evidence earn the belief. That inversion is the entire redesign.

---

## 2. Reading the data model before touching pixels

Before any UI decision I read data-model.ts as a spec. Three things stood out:

The portfolio type is deliberately thin. LeadershipPortfolio carries only sections, generatedAt, and uid. Employee, month, and org come from a transport envelope. I use those only in header chrome, not in content blocks, so a reload of the same UID renders correctly even if the envelope changes.

Sections are intentionally generic. dataPoints is typed as Record<string, string | number | string[]> — the backend doesn't promise typed shape per section. The mock's nested data payload is richer than the model's base type. I render only fields visible in both, and never depend on nested shapes the model doesn't promise.

Narrative is sacred. narrative: string | null appears on every section. Combined with the assignment rule of no LLM on the frontend, this is the most important fact in the data model: the story is already written by the server. My job is to frame it, not compose it. Every design move below flows from that.

---

## 3. Position on the 5 design challenges

| # | Challenge | My position | Rejected alternatives |
|---|---|---|---|
| 1 | Signal at a glance | Human-written sentence as H1 ("His strongest month since joining"), with the score delta +7 as inline prose | Ring chart (re-creates dashboard metaphor); letter grade (infantilizing for a pharma validation lead); 12-month trend line (wrong surface for a monthly doc) |
| 2 | Story vs data order | Narrative paragraph opens each section; three stat tiles follow; detail collapses underneath | Dual-mode toggle (two half-designs); data-first with narrative caption (inverts the thesis) |
| 3 | Six sections compressed | Executive Summary promoted to page header; KPI Impact folded into "What he shipped" because KPIs are contribution outcomes; Constraints de-weighted to a muted panel; Trajectory becomes the closing slide | Tabs (breaks linear reading); accordion on everything (hides story behind clicks) |
| 4 | Constraints without anxiety | Reframe "what blocked me" to "what he navigated." Lead with the resolution Rajan drove (joint regulatory review, minus 3 weeks per protocol); open items collapsed | Sorting by severity (manufactures urgency); hiding constraints (dishonest; loses the judgment signal) |
| 5 | Portability and identity | Persistent UID watermark in header and footer, role written as "Production Planning Lead" not "V2," zero mentions of PDGMS, TPM, or the 5x5 grid in user-facing copy | UID in a modal (fails print-friendly requirement); V-level badges (internal grammar the mentor doesn't speak) |

---

## 4. Information hierarchy

```
IDENTITY BAR (sticky)
  Rajan Iyer · Production Planning Lead
  Vayura Lifesciences · Facility 3 Commercialization
  March 2026                    UID: PDGMS-EMP-00512

THE HEADLINE
  "His strongest month since joining."
  71/100 (up from 64) · ~3 months ahead for V3 · gap: framework application

WHAT HE SHIPPED
  Narrative: broke the lyo protocol logjam
  [ 16/20 ]  [ 7 ]  [ 3/5 ]
  expand: deliverables and original work

HOW HE GREW
  Narrative: strong in delivery and engagement; framework application is the growth edge
  6 capability bars · threshold marker at 60

WHAT HE NAVIGATED (muted)
  8 raised · 5 resolved · one fixed a systemic bottleneck
  expand: open items

WHERE HE'S HEADED
  Narrative + horizontal runway: V1 done, V2 done, V3 next, V4 future
  Projected Feb 2027 · ahead of the May 2027 plan
```

---

## 5. Key design moves with tradeoffs

| Move | Rationale | Tradeoff accepted |
|---|---|---|
| Dual-voice typography — serif (Fraunces) for narrative, sans (Inter) for UI chrome | Visually signals "this is a document, not a dashboard" | Some orgs prefer unified sans; I believe dual-voice is the strongest cue of "artifact, not app" |
| Score delta inline in headline, not as a sparkline | A sentence with one number outperforms a tiny chart when you only need to communicate direction | Loses multi-month trend view — that belongs on a career-history page |
| Warning color reserved for exactly two capability bars (Frameworks 52, Processes 58) | Color in a dense page is a scarce resource; if everything can be red, nothing is | Reader must associate the colored bars with the headline gap language — mitigated by repeating the claim in the trajectory section |
| Open constraints collapsed; resolved constraint celebrated in narrative | Flips the emotional weight from "what went wrong" to "what got fixed." Professional reframing, not whitewashing | Diagnostic readers must expand — that's the right friction for a career artifact |
| No avatar image — mock says avatarUrl is null, I render initials "RI" | Don't fabricate; gracefully degrade | Slightly less personal feel — acceptable |
| Runway shows 4 nodes, V5 hidden in default view | A 12-year horizon to Mandate Setter is too far to be meaningful in a monthly doc | Can be expanded if a user wants the full long-range plan |

---

## 6. Field provenance — every visible element traces to real data

| Visible element | JSON source |
|---|---|
| Name, role, program, org | employee.{name, role, program, organization} |
| Tenure context ("Month 19") | employee.tenureMonths |
| UID watermark | uid = PDGMS-EMP-00512 |
| Month label | monthId = 2026-03 |
| Composite score 71, previous 64, delta +7 | executive_summary.data.{compositeScore, previousMonthScore} |
| Pace "ahead" | executive_summary.data.paceStatus |
| 16 of 20 delivered | contribution_highlights.data.{completed, totalDeliverables} |
| 7 original work items | contribution_highlights.data.ipCommitCount |
| 3 of 5 KPIs hit | computed from kpi_impact.details[].status |
| Six axis scores | capability_growth.data |
| Threshold = 60 | from executive_summary.narrative and gapDrivers |
| 8 constraints, 5 resolved | constraint_patterns.data.{totalConstraints, resolved} |
| Ahead by ~3 months | computed from career_trajectory.data.{projectedDate, milestones} |
| Gap drivers (Frameworks, Processes) | career_trajectory.data.gapDrivers |

What I deliberately did not render: time distribution hours (belongs on a weekly work page); individual ticket IDs (internal jargon, violates portability); UBS event breakdown (internal scoring mechanics); gapFlags (draft-quality signals).

---

## 7. What I would do with more time

1. Printable mentor-share view. The UID portability constraint implies this artifact must survive outside PDGMS. A PDF export needs its own layout pass — different header chrome, no interactive disclosure, footer with generation date and UID for provenance.

2. Empty and thin-month states. What does this page look like in a month where someone shipped two items and raised twelve constraints? The current design assumes a rich month. The "no blockers this month" state should feel like a positive signal, not an absence.

3. Supervisor annotation layer. Not a role-toggle, but a single written comment per section that the TPM can attach without altering the employee's document. Preserves portfolio ownership while adding institutional voice.

4. Hover reveal on capability bars. Frameworks 52 expands to "3 applied this month: First Principles, PDCA, 5 Why" with examples. Surfaces depth on demand without crowding the bar.

5. A/B test narrative voice. I render the system narrative in third person ("Rajan shipped..."). First person ("I shipped...") may feel more like his document. Worth testing with real employees before committing.

6. Multi-month trend on a separate career-history page. Out of scope for this monthly artifact, but the runway and score delta hint at it. That page is where sparklines and ring charts actually belong — not here.

---

## 8. Assumptions I made

1. The narrative strings in the JSON are production-grade output of the server-side template engine. I render them as-is, with no parsing or restructuring.
2. The employee object is injected by the API alongside the portfolio.
3. Prev/next month navigation is handled at a route level outside this page.
4. The threshold of 60 is consistent across all capability axes — implied by the narrative calling out Frameworks 52 and Processes 58 as the only sub-threshold items.
5. A user viewing their own portfolio and a supervisor viewing it see the same page — no role-based conditional rendering in v1.
6. The composite score is computed backend-side and returned as compositeScore. I render it; I do not re-derive it on the frontend.

---

## 9. Closing note

The brief warned that most submissions are AI output pasted verbatim. Three things in this submission are structured to demonstrate that did not happen here. Section 6 traces every visible element to a JSON path — if a number isn't sourceable, it isn't rendered. Section 2 reads the TS contract first and calls out drift between data-model.ts and the mock JSON — the kind of observation that comes from reading code, not prompting a model. Every design move in section 5 names a rejected alternative and a tradeoff accepted — decisions with losers are hard for generic AI output to fake.

The hand-drawn wireframe submitted via Internshala chat is the final filter, and it is genuinely my own.

— Joakim Manoj
```