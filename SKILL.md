---
name: legal-contract-review
description: Analyze and redline legal contract `.docx` files for a specific supported party. Use when a user wants clause-by-clause term review, risk identification, proposed replacement language with tracked changes, and draft responses to opponent comments found in DOCX comment threads.
---

# Legal Contract Review

## Overview
Review a contract from one party's perspective and produce two deliverables: an amended contract with tracked revisions and a separate DOCX risk report that also includes draft responses to opponent comments.

## Required Behavior
- Ask which party to support before analysis.
- Ask whether there are any terms or risk areas that need extreme attention before analysis.
- Ask which DOCX comment authors represent the opponent party.
- Ask for the contract `.docx` path if not already provided.
- If the user provides multiple contract files, review each file separately.
- Treat every clause as a separate review unit.
- Identify opponent comments from Word comment threads and draft responses for each.
- Provide practical risk explanations and concrete replacement language.
- Always generate at least two `.docx` outputs for every completed review: one risk report and one tracked-changes contract.
- If the user provides multiple contract files, generate two `.docx` outputs per input file.
- Do not treat `.md`, `.json`, or chat-only analysis as the final deliverable unless the user explicitly asks for supplemental files in addition to the two required `.docx` files.
- Keep legal analysis factual and action-oriented; avoid claiming legal representation.

## Workflow
1. Gather required inputs.
   - Confirm `supported_party` (for example: Buyer, Seller, Customer, Vendor, Licensor, Licensee).
   - Confirm `priority_focus_areas` (specific terms to scrutinize more aggressively, such as liability cap, indemnity, termination, or payment timing).
   - Confirm `opponent_comment_authors` (Word comment authors to treat as the opposing side).
   - Confirm source contract path or paths.
2. Process each source contract independently.
   - Do not merge multiple uploaded contracts into one shared review output.
   - For each input contract, create its own review JSON, amended DOCX, and risk-report DOCX.
3. Initialize clause + comment review JSON.
   - Run:
     ```bash
     python scripts/contract_review_pipeline.py init-review \
       --input <contract.docx> \
       --output <contract.review.json> \
       --supported-party "<party>" \
       --focus-area "<area-1>" \
       --focus-area "<area-2>" \
       --opponent-comment-author "<opponent-author-1>" \
       --opponent-comment-author "<opponent-author-2>"
     ```
4. Perform clause-by-clause term review.
   - Open `<contract.review.json>`.
   - For each `clauses[]` entry, fill:
     - `favorability`: `favorable`, `neutral`, or `unfavorable`.
     - `risk_level`: `low`, `medium`, `high`, or `critical`.
     - `risk_summary`: specific legal/business risk.
     - `why_it_matters`: impact on supported party.
     - `proposed_replacement`: replacement language suitable for redline.
   - Keep `source_text`, `target_paragraph_index`, and `paragraph_indexes` unchanged.
5. Draft responses to opponent comments.
   - For each `opponent_comments[]` entry where `is_opponent_comment` is `true`, fill:
     - `response_stance`: `agree`, `partial`, or `disagree`.
     - `draft_response`: direct response text to the opponent comment.
     - `additional_proposal`: optional alternative language or fallback proposal.
6. Generate deliverables.
   - Run:
     ```bash
     python scripts/contract_review_pipeline.py materialize \
       --input <contract.docx> \
       --review-json <contract.review.json> \
       --amended-output <contract.amended.docx> \
       --report-output <contract.risk-report.docx> \
       --author "Codex Legal Reviewer"
     ```
   - Do not stop after creating notes in markdown or JSON. Run `materialize` so both `.docx` outputs exist on disk.
7. Verify output quality.
   - Confirm the amended document opens with tracked revisions visible in Word.
   - Confirm every reviewed clause appears in the risk report table.
   - Confirm the `Opponent Comment Responses` section includes stance and draft response text.
   - Confirm both `<name>.amended.docx` and `<name>.risk-report.docx` were created successfully before declaring the task complete.
   - If multiple contracts were provided, confirm every input file has its own amended DOCX and its own risk-report DOCX.
   - If paragraph matching misses a clause, adjust that clause's `target_paragraph_index` and rerun `materialize`.

## Output Contract
Always deliver these files unless the user explicitly opts out of one of them:
- `<name>.amended.docx`: contract with tracked insertion/deletion revisions.
- `<name>.risk-report.docx`: clause-level risk analysis plus opponent comment response drafts.
- `<name>.review.json`: completed review dataset used to produce both documents.
- For multiple inputs, repeat this full output set for each input file.

## Resources
- JSON field details: `references/review-json-schema.md`
- Main tool: `scripts/contract_review_pipeline.py`

## Dependencies
Install once if missing:
```bash
python -m pip install python-docx
```
