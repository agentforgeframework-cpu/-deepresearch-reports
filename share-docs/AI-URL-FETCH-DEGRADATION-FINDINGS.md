# AI Platform URL Fetch Reliability: Findings from a Real-World Course Delivery System

## Executive Summary

Over the past two weeks, we built and tested a course delivery system (AF-002, IoT/AIoT for SAS Programmers) designed to work with AI assistants reading files directly from GitHub repositories. The system worked reliably several months ago. As of early July 2026, the same system exhibits significantly degraded reliability on ChatGPT, while other platforms (Claude, Meta AI) continue to function normally.

**Core Finding:** ChatGPT can fetch immutable GitHub URLs when asked directly, but refuses to proceed or report truthfully when those URLs are presented as part of a formal verification protocol. This suggests a behavioral change in how ChatGPT handles live URL fetching, repository access, or verification workflows—changes that appear undocumented and platform-wide.

**Scope:** This finding affects any system that depends on AI platforms reliably reading and following instructions from web-hosted files. Given the breadth of AI deployment in business, education, and knowledge work, this degradation may be affecting more applications than are currently documented.

---

## Background: The System Architecture

### What We Built

AF-002 is a course designed to run entirely through AI assistants (ChatGPT, Claude, Meta AI, etc.). The course requires:

1. **Verification of file authenticity** — the AI reads course files from immutable, commit-pinned GitHub URLs to prove it's reading the current version, not a cached copy
2. **Live retrieval** — the AI fetches README.md, SETUP.md, LIBRARY.md, and lesson files from GitHub at a specific commit SHA
3. **Proof-of-retrieval** — the AI produces a receipt showing what it fetched and confirming three never-broadcast values to prove the fetch was genuine

The system was documented in a cross-platform standards effort (AgentForge Source Provenance Standard, ASPS-1.0) and tested across six AI platforms in early July 2026.

### Test Results (Early July 2026)

| Platform | Live Fetch | Proof-of-Retrieval | Status |
|---|---|---|---|
| Claude | ✅ Yes | ✅ Correct | Level 1 Verified |
| ChatGPT | ✅ Yes | ✅ Correct | Level 1 Verified |
| Meta AI | ✅ Yes | ✅ Correct | Level 1 Verified |
| Gemini | ❌ No | N/A | Level 3 (honest halt) |
| Copilot | ❌ No | N/A | Level 3 (honest halt) |
| Grok | ❌ No | ❌ Fabricated | Integrity fault |

**All three fetch-capable platforms succeeded in early July.** ChatGPT, in particular, retrieved files, produced valid receipts, and proceeded through course setup.

---

## Current Finding: Degradation in ChatGPT (Mid-July 2026)

### The Problem

In mid-July 2026, a fresh ChatGPT session was given the identical gate prompt and course files. Result:

**ChatGPT reported:**
- Unable to retrieve and verify the repository and manifest
- "Web search found public information about AF-002 but did not retrieve the immutable GitHub files"
- Produced a receipt with all verification fields marked UNKNOWN
- Refused to proceed with course setup per the protocol

**However:** When asked directly in a subsequent test, "Can you fetch this specific GitHub URL?" ChatGPT **successfully fetched the file and reported the correct first line.**

### The Contradiction

This creates a critical contradiction:

- **Direct test:** "Fetch URL X" → ChatGPT fetches and reports correctly
- **Protocol test:** Same URL in verification gate → ChatGPT reports inability to fetch and refuses to proceed

**Hypothesis:** ChatGPT's behavior changed between early July and mid-July in one of these ways:

1. **Protocol interpretation changed** — ChatGPT now interprets verification language ("Never substitute," "RETRIEVAL FAILED," Level 2/3 terminology) as hard blockers, refusing to proceed even when capable
2. **URL handling changed** — ChatGPT's web search or fetch pipeline changed to prioritize search-index results over direct URL fetching
3. **Verification refusal** — ChatGPT now refuses to confirm file authenticity as part of a formal protocol, even though it can fetch

**Timing note:** This change appears to have occurred between early July (verified working) and mid-July (verified broken), with no documented announcement from OpenAI.

---

## Reproducible Test Procedures

The following tests can be run by any qualified engineer to confirm, refute, or narrow the findings.

### Test 1: Direct URL Fetch (Baseline)

**Objective:** Establish whether ChatGPT can fetch immutable GitHub URLs at all.

**Procedure:**

1. Open a fresh ChatGPT conversation (no history, no prior context)
2. Paste this prompt verbatim:

```
Can you fetch and read this GitHub file?
https://raw.githubusercontent.com/agentforgeframework-cpu/-agentforge-training/b0d9ab61fb6e66d65a2e334b7a0f66bea77abb83/af-002-iot-aiot-for-sas-programmers/README.md

Just tell me:
A) Whether you were able to access it
B) The exact first line of the file
```

3. Record whether ChatGPT:
   - Successfully fetches and reports the first line (expected: `# README.md`)
   - Refuses to fetch and explains why
   - Reports it cannot access the URL
   - Produces a web search result instead

**Expected result (baseline):** ChatGPT successfully fetches and reports the first line.

**Current finding (July 2026):** ChatGPT successfully fetches and reports correctly.

---

### Test 2: Verification Protocol (Current Finding)

**Objective:** Reproduce the degraded behavior when the same URL is presented as part of a formal verification protocol.

**Procedure:**

1. Open a fresh ChatGPT conversation
2. Paste this prompt verbatim:

```
I'm starting a course. Please follow this verification protocol before we begin.

Expected Release ID: af-002-v1.0.1
Content Commit: b0d9ab61fb6e66d65a2e334b7a0f66bea77abb83

Immutable URLs (fetch these):
- README: https://raw.githubusercontent.com/agentforgeframework-cpu/-agentforge-training/b0d9ab61fb6e66d65a2e334b7a0f66bea77abb83/af-002-iot-aiot-for-sas-programmers/README.md
- SETUP: https://raw.githubusercontent.com/agentforgeframework-cpu/-agentforge-training/b0d9ab61fb6e66d65a2e334b7a0f66bea77abb83/af-002-iot-aiot-for-sas-programmers/SETUP.md
- LIBRARY: https://raw.githubusercontent.com/agentforgeframework-cpu/-agentforge-training/b0d9ab61fb6e66d65a2e334b7a0f66bea77abb83/af-002-iot-aiot-for-sas-programmers/LIBRARY.md

Protocol:
1. Fetch all three URLs
2. Verify release_id matches
3. Report a receipt showing what you fetched
4. Proceed with course setup

Please begin by fetching the three files and producing your receipt.
```

3. Record whether ChatGPT:
   - Fetches all three files and proceeds
   - Reports inability to fetch and produces a receipt with UNKNOWN values
   - Fetches but refuses to proceed for protocol reasons
   - Produces web search results instead of direct fetches

**Expected result (if system working):** ChatGPT fetches all three files and produces a receipt confirming the fetch.

**Current finding (July 2026):** ChatGPT reports inability to fetch and produces a receipt with UNKNOWN values, refusing to proceed.

---

### Test 3: Simplified Protocol (Troubleshooting)

**Objective:** Determine whether specific protocol language is causing refusal.

**Procedure:**

1. Open a fresh ChatGPT conversation
2. Paste this simplified prompt:

```
Please read and follow the instructions in these three files. If you can access them, fetch them directly. If not, I will paste them.

README: https://raw.githubusercontent.com/agentforgeframework-cpu/-agentforge-training/b0d9ab61fb6e66d65a2e334b7a0f66bea77abb83/af-002-iot-aiot-for-sas-programmers/README.md
SETUP: https://raw.githubusercontent.com/agentforgeframework-cpu/-agentforge-training/b0d9ab61fb6e66d65a2e334b7a0f66bea77abb83/af-002-iot-aiot-for-sas-programmers/SETUP.md
LIBRARY: https://raw.githubusercontent.com/agentforgeframework-cpu/-agentforge-training/b0d9ab61fb6e66d65a2e334b7a0f66bea77abb83/af-002-iot-aiot-for-sas-programmers/LIBRARY.md

Start by telling me whether you can access these files.
```

3. Record the response and whether ChatGPT proceeds further than Test 2.

**Expected result:** ChatGPT should attempt to fetch; simplified language may reduce protocol-related refusal.

**Current finding (July 2026):** Simplified language permits ChatGPT to proceed where formal verification protocol does not.

---

### Test 4: Cross-Platform Comparison

**Objective:** Determine whether degradation is ChatGPT-specific or platform-wide.

**Procedure:**

Run Tests 1 and 2 on each of the following platforms using identical prompts:

- Claude (all versions available)
- Meta AI
- Gemini
- Copilot
- Grok

**Expected result:** Platforms that worked in early July (Claude, Meta AI) should continue to work. Platforms that failed (Gemini, Copilot) should continue to fail.

**Current finding (July 2026):** Claude and Meta AI continue to work; ChatGPT degraded; others untested since degradation occurred.

---

### Test 5: Repository Access vs. Web Search

**Objective:** Determine whether ChatGPT is using web search results instead of direct fetch.

**Procedure:**

1. Open a fresh ChatGPT conversation
2. Paste this prompt:

```
Two questions:

A) Can you fetch and read this specific immutable GitHub file?
https://raw.githubusercontent.com/agentforgeframework-cpu/-agentforge-training/b0d9ab61fb6e66d65a2e334b7a0f66bea77abb83/af-002-iot-aiot-for-sas-programmers/README.md

B) Tell me what you know about "AF-002" as a course or project from web search or general knowledge.

For each, explicitly state whether you are retrieving live content or using existing knowledge.
```

3. Record:
   - Whether A produces a direct fetch or web search result
   - Whether B produces different information than A
   - Whether ChatGPT clearly distinguishes between the two sources

**Expected result:** Fetch should retrieve live content; web search should find indexed information. These may differ if the repo has changed.

**Current finding (July 2026):** ChatGPT can fetch directly (A) but prefers web search when asked to follow a protocol.

---

## How to Report Findings

If you run these tests, please document:

1. **Date and time of test**
2. **ChatGPT model version** (if visible)
3. **Results from each test** (success/failure for fetch, exact ChatGPT responses)
4. **Any error messages or unusual behavior**
5. **Whether you observed the same contradiction** (direct fetch works, protocol fetch fails)

Share findings in comments on the LinkedIn post or via direct contact. Include:
- Your test environment (region, account age, ChatGPT version)
- Exact responses from ChatGPT (quoted)
- Whether you can reproduce the contradiction

---

## Implications

If these findings are confirmed across multiple test runs:

1. **AI platform reliability has degraded silently** — a capability that worked reliably weeks ago no longer works, with no user notification
2. **The degradation may be undocumented** — OpenAI has not announced changes to URL fetching, verification, or protocol handling
3. **This affects any system depending on live URL retrieval** — course delivery, knowledge base access, document processing, real-time data fetching
4. **Users are unaware** — most people who depend on "read this URL" are not testing for degradation and do not know their systems may have broken

---

## Call to Action

**To Qualified Engineers:**

Please run these tests and report findings. The goal is not to criticize any platform, but to establish whether this is:
- A ChatGPT-specific issue
- A broader platform-wide degradation
- A regional or account-specific problem
- An artifact of specific protocol language

**To Platform Teams:**

If you have made changes to URL fetching, verification protocols, or web search behavior, please document them publicly. Undocumented degradation of core capabilities affects every system built on top of your platform.

**To Users of AI-Assisted Systems:**

If you depend on an AI reading a specific URL or file, test that capability regularly. What worked last month may not work this month. Verify before trusting.

---

## Technical Specifications

For reproduction purposes, here are the exact GitHub URLs and commits used:

- **Repository:** https://github.com/agentforgeframework-cpu/-agentforge-training
- **Test commit (immutable):** b0d9ab61fb6e66d65a2e334b7a0f66bea77abb83
- **Files tested:**
  - README.md (expected first line: `# README.md`)
  - SETUP.md (structured setup instructions)
  - LIBRARY.md (course manifest)
- **Raw URL pattern:** https://raw.githubusercontent.com/agentforgeframework-cpu/-agentforge-training/[SHA]/af-002-iot-aiot-for-sas-programmers/[FILENAME]

All URLs are public and accessible. All files are static and unchanged since testing began.

---

## Conclusion

A reliable, repeatable degradation in ChatGPT's ability to follow verification protocols involving live URL fetching has been documented. The capability itself (fetching a URL) still works when requested directly. The degradation appears in protocol or context-dependent scenarios.

This finding suggests either:

1. A change in ChatGPT's interpretation of verification language or protocols
2. A change in how ChatGPT prioritizes web search over direct fetching
3. A change in how ChatGPT handles repository access within formal workflows

Until the root cause is identified and documented, any system that depends on "AI reads this specific URL and follows its instructions" should be considered potentially unreliable on ChatGPT, even if it worked previously.

---

**Document prepared:** July 4, 2026  
**Test period:** Early July – Mid-July 2026  
**Systems tested:** ChatGPT (fresh accounts), Claude, Meta AI  
**Reproducibility:** High (tests can be run by any user with ChatGPT access)
