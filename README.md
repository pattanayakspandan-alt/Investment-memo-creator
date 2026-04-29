# Investment Memo Skill

A Claude skill for early-stage VC deal evaluation at Draper Associates. Given a founder call transcript, pitch deck, or one-pager, it produces a post-call email to the principal and/or a full formatted investment memo (.docx) following the DA diligence template.

---

## What It Produces

| Output | Format | When to Use |
|--------|--------|-------------|
| Post-call email | Rendered in chat via `message_compose` | Same-day, after any founder call |
| Full investment memo | `.docx` (DA template) | Before IC, after first or second call |
| Intermediate research docs | `.md` (3 files) | For audit trail and deeper review |

When triggered, the skill asks which output you need before doing any work:
- **Post-call email only** -- fast path, ~10 min
- **Full memo only** -- deep path, 3-stage pipeline
- **Both** -- email first, then memo; research is shared across both so nothing runs twice

---

## Inputs

Upload files directly into the Claude chat before triggering the skill. The more context provided, the sharper the output.

### Core Inputs (provide at least one)

| Input | Format | Notes |
|-------|--------|-------|
| Founder call transcript | `.docx` or `.txt` (Otter.ai export) | Most valuable input. Claude extracts every founder claim and verifies each one against public sources independently. |
| Pitch deck | `.pdf` or `.pptx` | Used for product, market, and traction context. |
| One-pager or intro email | `.pdf`, `.docx`, or pasted text | Useful for pre-call context or inbound deal briefs. |

### Metadata Claude Asks For

The skill prompts for these upfront before starting any research:

**For the email:**
- Principal name (e.g., Tim)
- Company name and website URL
- Stage (Pre-seed, Seed, Series A, etc.)

**For the full memo (additional):**
- Analyst name
- Sector / category
- Referral source (who introduced the deal)
- Zoom / recording link (optional)
- Data room link (optional)
- Specific concerns or focus areas (optional)

---

## How It Works

### Post-Call Email (Path A)

1. Reads all uploaded materials
2. Runs targeted web searches to verify and supplement transcript claims: founder backgrounds, funding history, competitors, market sizing, public red flags
3. Drafts a 500-800 word email structured as:
   - **Recommendation first** (Pass / Proceed to second call / Fast-track to IC) with 2-3 sentence rationale
   - What they do and why now
   - The founders (background, motivation)
   - Market and size
   - Moat and competitors
   - Traction and growth plan
   - Funding history and current ask
   - Analyst notes (red flags, call observations, open diligence items)
4. Scores applicable dimensions inline: Problem/Timing [X/5], Founders [X/5], Market [X/5], Moat [X/5], Traction [X/5]
5. Flags all externally sourced data with [per Source] notation
6. Presents via `message_compose` -- one variant for clear calls, two variants (bull/bear) for borderline ones

### Full Investment Memo (Path B)

The memo runs a structured 3-stage diligence pipeline before generating the .docx:

**Stage 1 -- Identity Lock and Fact Gathering**
Confirms company identity across multiple sources, extracts all founder claims from the transcript tagged `[FD]`, and gathers raw facts across: corporate info, funding history, team, product, technology, IP, traction, customer feedback, press, and job postings. Every claim is tagged with a data confidence tag and cited. A gap register logs everything that could not be found.

**Stage 2 -- Deep Analysis**
Verifies each `[FD]` claim against public sources. Claims are promoted to `[V]` (verified) or flagged `[FD-]` (contradicted). Produces analytical verdicts across: team, problem/solution, product/tech, IP, market, competition, business model, traction, customer feedback, GTM, and press.

**Stage 3 -- Risk Synthesis and Recommendation**
Consolidates findings into a risk register, investment thesis (bull case, bear case, comparable exits), 12-15 open questions for the next call, a 200-250 word executive summary, and a final recommendation.

**Score Review**
After the pipeline, Claude proposes scores (1-5) across 8 dimensions with one-line justifications. The analyst reviews and can adjust before the memo is generated.

**Memo Generation**
The final .docx is generated from the synthesis. It is tag-free (all data classification tags stripped), formatted to the DA template, and includes gray-italic notices for any sections with missing data.

### Intermediate Outputs

Three markdown files are saved alongside the final memo:
- `[Company]_Stage1_FactGathering.md` -- raw facts with data tags and citations
- `[Company]_Stage2_DeepAnalysis.md` -- analytical verdicts per dimension
- `[Company]_Stage3_Synthesis.md` -- risk register, thesis, open questions, recommendation

These are the audit trail. They retain all data classification tags so the analyst can see exactly what was verified vs. inferred vs. founder-disclosed.

---

## Memo Format

The final .docx follows the DA investment memo template with 17 sections:

1. Title page with analyst metadata and company website screenshots
2. Executive Summary (light blue callout box)
3. Investment Terms (data table)
4. Overall Evaluation -- scores 1-5 across 8 dimensions
5. Company Overview
6. Founding Team (founder comparison table, stakeholder connections)
7. Product and Technology (problem/solution callout, product image)
8. Market, Competition and Regulatory Landscape (competitor table)
9. Business Model and Strategy
10. Company Risks
11. Traction and Validation
12. Financial Snapshot
13. Fundraising and Deal Terms
14. Milestones Roadmap and Use of Funds
15. Key Opinion Leaders
16. Investors Commentary
17. Customer Commentary
18. Follow-up Questions and Next Steps

**Formatting:** Dark navy (#13305f) section headers, light blue (#cfe2f3) callout boxes, Arial font throughout, US Letter page size.

**Empty sections** are kept with a notice: *"No information available. To be completed during due diligence."* Nothing is silently omitted.

---

## Scoring Rubric

Scores (1-5, half-point increments) are proposed across 8 dimensions. The overall score is not a simple average -- it is stage-weighted with heavier weight on team and product at seed, and traction and financials at Series A+.

| Dimension | What It Assesses |
|-----------|-----------------|
| Founders and Team | Domain expertise, founder-market fit, team composition, prior track record |
| Product and Tech | Maturity, technical moat, IP, scalability, API dependency risk |
| Market, Competition and Regulation | TAM credibility, competitive positioning, regulatory environment, timing |
| Business Model, Strategy and Vision | Revenue model, unit economics signals, GTM motion, long-term vision |
| Traction and Validation | Revenue, growth trajectory, customer quality, retention signals |
| Financials, Fundraising and Deal Terms | Valuation vs. traction, cap table quality, deal terms |
| Key Opinion Leaders | Quality and relevance of KOL relationships |
| Co-investors | Lead investor quality, syndicate composition |

Score interpretation: 4.5-5.0 = Fast-track, 3.5-4.0 = Proceed, 2.5-3.0 = Soft Pass, 1.0-2.0 = Hard Pass. A single overriding factor (e.g., legal red flag) can override the numeric score.

---

## Data Classification Tags (Intermediate Docs Only)

The intermediate stage docs use a tagging system to distinguish confidence levels. These tags do not appear in the final memo.

| Tag | Meaning |
|-----|---------|
| `[V]` | Verified -- directly stated in a citable, timestamped public source |
| `[I]` | Inferred -- reasonably derived from signals; derivation shown |
| `[S]` | Speculated -- analytical judgment, no direct evidence |
| `[FD]` | Founder Disclosed -- stated by the founder in the transcript; unverified |
| `[FD+]` | Founder claim partially supported by public evidence |
| `[FD-]` | Founder claim contradicted by public evidence -- flagged as red flag |
| `[NF]` | Not Found -- searched and unavailable |
| `[ST]` | Stale -- source older than 12 months |
| `[U]` | Unverified -- source exists but credibility is unclear |

---

## Key Principles

**Founder claims are hypotheses, not facts.** Every claim from the transcript is tagged `[FD]` and independently verified before appearing in any output. If verification fails, it is flagged, not silently included.

**Recommendation leads every output.** Both the email and the memo lead with the verdict. The principal gets the crux before the detail.

**Flag gaps explicitly.** Sections with missing data are marked rather than omitted. "Not found" is more useful than silence.

**No promotional language.** All founder and company marketing copy is rewritten in neutral analytical tone.

**Source precision.** Docket activity, funding figures, and traction metrics are not inferred from ambiguous snippets. Uncertainty is flagged explicitly.

**No em dashes.** Commas, semicolons, or periods are used throughout all outputs.

---

## File Structure

```
investment-memo/
├── README.md                        -- this file
├── SKILL.md                         -- main skill instructions and workflow
├── references/
│   ├── pipeline-prompts.md          -- 3-stage diligence pipeline methodology
│   ├── memo-format.md               -- section-by-section memo format spec
│   ├── scoring-rubric.md            -- 1-5 scoring framework per dimension
│   └── post-call-email.md           -- post-call email structure and tone spec
└── scripts/
    └── generate_memo.js             -- Node.js docx-js memo generation template
```

---

## Best Practices

**Always upload the Otter.ai transcript if you have one.** It is the single most valuable input. Without it, the skill relies on the deck and web research, which produces a thinner output with less founder claim verification.

**Share specific concerns upfront.** If something felt off on the call or a particular dimension needs scrutiny, say so before the skill starts. It will weight those areas more heavily.

**Review proposed scores before approving memo generation.** After the pipeline runs, Claude pauses and shows proposed scores with one-line justifications. This takes 30 seconds and directly affects the executive summary and overall recommendation.

**For borderline calls, read both email variants.** The bull and bear framings are written to help the analyst recognize which read on the deal is more defensible, not just to offer options.

**Treat gray-italic text in the memo as your diligence checklist.** Any field that appears in gray italics is a gap identified during research. Bring those into the next call or assign them to the team.

**Sanity-check externally sourced data on sensitive deals.** Claude flags web-sourced data throughout, but recent funding rounds, litigation history, and regulatory status can change quickly. Verify independently before a memo goes to IC.

---

## Limitations

- Website screenshots require the `zyte-screenshots` skill to be installed. Without it, placeholders are inserted and the analyst is asked to add screenshots manually.
- Funding figures from Crunchbase and PitchBook are estimates; treat them as signals, not facts.
- The skill cannot access private data rooms, cap tables, or documents behind login walls.
- Context from very long transcripts (90+ min calls) may be partially summarized; the full transcript is always retained for reference.
- The skill does not send the email or submit the memo anywhere. All outputs are delivered to the analyst for review before use.
