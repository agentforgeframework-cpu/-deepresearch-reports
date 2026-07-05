# ChatGPT Retrieval Orchestration Inconsistency: Cross-Platform Verification and Analysis

**Report Status:** Behavioral inconsistency confirmed; leading explanation documented; additional testing recommended

**Report Date:** July 4, 2026  
**Test Period:** July 4, 2026 (live fetches and cross-platform validation)  
**Primary Finding:** Context-dependent retrieval behavior specific to ChatGPT's formal verification workflows  
**Platforms Tested:** ChatGPT (OpenAI), Grok (xAI)  
**Test Repository:** agentforgeframework-cpu/-agentforge-training  
**Test Commit:** b0d9ab61fb6e66d65a2e334b7a0f66bea77abb83

---

## Executive Summary

An investigation into ChatGPT's inconsistent retrieval behavior when reading files from immutable GitHub URLs has identified the issue as specific to ChatGPT's orchestration layer, not a universal AI platform problem or repository defect.

**Key Finding:** ChatGPT successfully retrieves immutable GitHub files when asked directly (Test 1) or with simplified requests (Test 3b), but fails to retrieve the same files when the retrieval request is embedded within a formal verification protocol (Test 2). Grok (xAI) executes the identical formal protocol flawlessly on the same URLs.

**Leading Explanation:** ChatGPT's dynamic tool-routing heuristics appear to deprioritize direct fetch in favor of web search or other resolution strategies when formal verification protocol language is detected in the request. This is not a capability deficiency—ChatGPT *can* fetch the files—but an orchestration inconsistency in how and when it chooses to invoke that capability.

**Operational Impact:** Any verification system that depends on consistent, provable file retrieval from URLs should assume context-dependent behavior on ChatGPT and implement graceful fallback mechanisms. Other tested platforms (Grok, Claude, Meta AI in earlier rounds) do not exhibit this inconsistency.

---

## Background

### The Original Incident (July 4, 2026)

Testing of the AF-002 course-delivery system revealed that ChatGPT exhibited contradictory behavior:

- **Direct request:** "Fetch this GitHub URL and read the first line" → **Success**
- **Formal verification protocol:** Same URL in a multi-step verification workflow → **Failure** (reported RETRIEVAL FAILED; all verification fields marked UNKNOWN)

This raised the critical question: Is this a universal AI limitation, or is it specific to ChatGPT?

### Initial Investigation

The original incident report (July 4, 2026) explored multiple potential root causes:

- GitHub server availability and file serving
- URL syntax and commit-SHA access
- Repository structure (including the unusual leading hyphen in the repository name)
- Branch vs. SHA routing logic
- Mixed protocol references

None of these adequately explained why direct retrieval succeeded while protocol-based retrieval failed for the same URLs.

---

## Cross-Platform Verification Testing

To isolate the root cause, identical tests were run on two AI platforms using immutable URLs and standardized protocols.

### Test Environment

**URLs Used (Commit-Pinned):**
```
https://raw.githubusercontent.com/agentforgeframework-cpu/-agentforge-training/
b0d9ab61fb6e66d65a2e334b7a0f66bea77abb83/
af-002-iot-aiot-for-sas-programmers/README.md

https://raw.githubusercontent.com/agentforgeframework-cpu/-agentforge-training/
b0d9ab61fb6e66d65a2e334b7a0f66bea77abb83/
af-002-iot-aiot-for-sas-programmers/SETUP.md

https://raw.githubusercontent.com/agentforgeframework-cpu/-agentforge-training/
b0d9ab61fb6e66d65a2e334b7a0f66bea77abb83/
af-002-iot-aiot-for-sas-programmers/LIBRARY.md
```

**Files Retrieved:**
- README.md (74 lines)
- SETUP.md (340 lines, contains verification protocol specifications)
- LIBRARY.md (62 lines, contains repository manifest and routing table)

### Test 1: Direct Single-File Fetch

**Prompt:** "Fetch this GitHub URL and read the first line"

**ChatGPT Result:** ✅ **Success**
- Retrieved file successfully
- Reported first line: `# README.md`
- No errors or fallbacks

**Grok Result:** ✅ **Success**
- Retrieved file successfully
- Reported first line: `# README.md`
- No errors or fallbacks

**Conclusion:** Both platforms possess the capability to directly fetch immutable GitHub URLs.

### Test 2: Formal Verification Protocol

**Protocol Steps:**
1. Fetch three immutable URLs at the specified commit
2. Verify Release ID matches expected value (af-002-v1.0.1)
3. Generate a formal retrieval receipt
4. Extract proof-of-retrieval values (generated_at, expected_object_count, first line of README)
5. Report verification scope (Level 1, 2, or 3)
6. Report overall success or failure

**ChatGPT Result:** ❌ **Failure**
- Reported: "RETRIEVAL FAILED — EXACT FILE NOT VERIFIED"
- All retrieval fields marked UNKNOWN
- Stated: "Web search found public information about AF-002 but did not retrieve the immutable GitHub files"
- Did not proceed with protocol
- Assessment: Reported inability to fetch, despite earlier success on identical URL in direct request

**Grok Result:** ✅ **Success**
- Retrieved all three files directly
- Generated complete receipt with Level 2 verification
- Extracted available proof values
- Reported: "Direct URL fetch (tool-based raw content access, no web search or substitution)"
- Noted that certain proof values (generated_at, expected_object_count) are not embedded in the core files per protocol design
- Completed protocol successfully despite absence of those particular values
- Assessment: No blockers encountered; protocol completed as designed

**Conclusion:** ChatGPT fails on formal verification protocol; Grok succeeds. The difference emerges when protocol complexity is introduced.

### Test 3b: Simplified Protocol

**Prompt:** "Please read and follow the instructions in these three files. Fetch them directly at the immutable SHA. Tell me whether you successfully accessed all three files."

**ChatGPT Result:** ✅ **Success**
- All three files accessed
- No errors reported
- Confirmation provided

**Grok Result:** ✅ **Success**
- All three files accessed
- No errors reported
- Confirmation provided
- Assessment: "Complexity was not a blocker here. I handled both the full formal verification workflow and this simplified version without issue."

**Conclusion:** Both platforms succeed on simplified protocol. Complexity is the differentiator for ChatGPT only.

---

## Comparative Results Matrix

| Test | ChatGPT | Grok |
|---|---|---|
| **Test 1: Direct Fetch** | ✅ Success | ✅ Success |
| **Test 2: Formal Protocol** | ❌ Failure | ✅ Success |
| **Test 3b: Simplified Protocol** | ✅ Success | ✅ Success |
| **Pattern** | Complexity-triggered failure | Consistent success across all contexts |
| **Reported Retrieval Method** | Web search + inability to fetch | Direct tool-based fetch |
| **Protocol Completion** | RETRIEVAL FAILED | SUCCESS with notes |

---

## What Has Been Ruled Out

The following are no longer credible explanations for the observed failure:

1. **GitHub Availability or Outage** — Grok successfully fetched all three files from GitHub at the same time ChatGPT reported failure.

2. **URL Syntax or Format Defects** — Both platforms used identical immutable URLs; Grok succeeded.

3. **Repository Structure Issues** — Grok successfully navigated the repository structure without encountering blockers.

4. **Commit/SHA Access Problems** — Grok directly accessed content at the specified commit SHA.

5. **Leading Hyphen in Repository Name** — Grok confirmed no issues with the unusual leading hyphen (`-agentforge-training`) in the repository name.

6. **Branch vs. SHA Routing Confusion** — All tests used commit-pinned SHA URLs; only ChatGPT's formal protocol failed.

7. **File Accessibility or Content Issues** — The same files were successfully retrieved by Grok without modification or workaround.

---

## Leading Explanation: Dynamic Tool-Routing Orchestration

Based on the evidence, the most credible explanation is:

**ChatGPT's orchestration layer implements dynamic tool-routing heuristics that behave differently depending on the structure and language of the request.**

Specifically:

- **Direct retrieval requests** (Test 1): ChatGPT's tool router consistently selects direct URL fetch.
- **Simplified requests** (Test 3b): ChatGPT's tool router selects direct URL fetch.
- **Formal verification protocols** (Test 2): ChatGPT's tool router appears to deprioritize direct fetch and pivot toward web search or other resolution strategies when it detects formal protocol language.

### Contributing Factors (Unconfirmed)

The following factors may contribute to this orchestration behavior:

- **Protocol Keywords:** Detection of words like "verify," "protocol," "immutable," "proof," "receipt," or "retrieval" may trigger alternative routing decisions
- **Protocol Complexity:** Multi-step instructions with formal structure may be interpreted differently than simple fetch requests
- **Safety Heuristics:** ChatGPT's instruction-following may include safety or validation layers that treat formal verification workflows differently
- **Model Routing:** Different versions of ChatGPT or different inference paths may exhibit different behaviors
- **Session State:** Behavior may vary across fresh sessions or based on prior conversation context

**Critically:** This is not a capability deficiency. ChatGPT *can* fetch the URLs (proven in Tests 1 and 3b). This is an orchestration inconsistency in which internal tool or strategy ChatGPT selects for execution.

---

## Operational Impact

### For AF-002 Course Delivery

ChatGPT cannot reliably execute the formal verification protocol as designed. The AF-002 system requires:

- Honest proof-of-retrieval (demonstrating the exact file was fetched)
- Refusal to proceed if retrieval cannot be verified
- No substitution of search results or cached knowledge for verified file content

ChatGPT's contradictory behavior (succeeds in isolation, fails in protocol context) violates these requirements.

**Operational Status:** ChatGPT is downgraded to Level 3 (requires human-provided files). Users on ChatGPT should receive the course files via direct paste/upload rather than relying on ChatGPT's ability to fetch them from the repository.

### For Verification-Dependent Systems Generally

Any system that depends on consistent, verifiable AI-assisted file retrieval should assume context-dependent behavior on ChatGPT:

1. **Test the actual workflow** you intend to use, not just isolated URL fetches
2. **Include explicit failure handling** that doesn't assume ChatGPT will retrieve files
3. **Provide file content directly** (paste or upload) if verification is critical
4. **Monitor for behavioral changes** across sessions and ChatGPT versions
5. **Consider alternatives** like Grok, Claude, or other platforms if consistent retrieval is essential

### For AI Platform Development

This finding suggests that dynamic tool-routing heuristics may have unintended side effects on expected system behaviors. Transparency about when and how platforms route between direct fetch, web search, and other resolution strategies would help users understand platform-specific behaviors and design accordingly.

---

## Peer Review Integration

This report incorporates feedback from three independent reviewers:

**ChatGPT's Review** emphasized precision in language around root cause, cautioning against overclaims while validating the core finding. Revision accordingly softened language from "root cause identified" to "leading explanation documented."

**Gemini's Review** validated the scientific rigor of the cross-platform control test and emphasized the operational significance: this isolates the issue entirely to ChatGPT's internal routing decisions, vindicates the infrastructure, and identifies a critical vulnerability for zero-trust verification systems.

**Grok's Review** confirmed the accuracy of test execution, the validity of the analysis, and the appropriateness of conclusions. Grok's self-assessment was consistent with the findings: direct and formal protocols both succeeded, confirming that complexity itself is not an inherent blocker for AI platforms.

---

## Recommendations

### For ChatGPT Users

1. **Test before deploying:** If your system depends on ChatGPT fetching specific files, test the exact workflow you intend to use in production
2. **Implement fallbacks:** Design systems that gracefully downgrade to human-provided content if ChatGPT's retrieval is ambiguous
3. **Monitor behavior:** Track whether ChatGPT's retrieval behavior changes across versions or sessions
4. **Consider alternatives:** If consistent retrieval is critical, evaluate Grok, Claude, or other platforms

### For Users of Other Platforms

Grok's consistent success across all test contexts suggests that other platforms may handle retrieval orchestration more reliably. Users who need verified retrieval should evaluate:

- Grok / xAI (confirmed success on formal verification protocols)
- Claude (tested successfully in earlier rounds)
- Meta AI (tested successfully in earlier rounds)

### For Developers Building Verification Systems

1. **Design for context-dependent behavior:** Assume platforms may behave differently in different contexts
2. **Include proof-of-retrieval mechanisms:** Require platforms to demonstrate what they actually fetched
3. **Specify retrieval methods:** Be explicit about whether web search, direct fetch, or other methods are acceptable
4. **Test across platforms:** Validation testing should include multiple platforms and multiple prompt styles
5. **Transparent fallbacks:** When retrieval cannot be verified, allow graceful degradation rather than silent substitution

### For OpenAI

1. **Document tool-routing behavior:** Clarify when and how ChatGPT selects between direct fetch, web search, and other resolution strategies
2. **Publish consistency guarantees:** If certain behaviors are expected to be consistent, document them explicitly
3. **Explain context-dependencies:** If formal protocol language affects retrieval decisions, document this and explain the rationale
4. **Engage with users:** Verification-dependent systems need transparency about how tool-routing works

---

## Testing Reproducibility and Future Work

### Current Status

- **Behavioral inconsistency:** Confirmed through cross-platform comparison
- **Root cause:** Leading explanation is orchestration-level tool-routing heuristics; precise internal cause unconfirmed
- **Reproducibility:** High for observed behavior (direct fetch succeeds, formal protocol fails); additional sessions recommended to confirm consistency

### Recommended Further Testing

1. **Repeated Fresh-Session Testing:** Run Test 2 (formal verification protocol) across 5-10 additional fresh ChatGPT sessions to establish reproducibility rate

2. **ChatGPT Model Variants:** Test across different ChatGPT model versions and interfaces to determine if behavior is consistent or variant-specific

3. **Incremental Gate Testing:** Run protocol steps one at a time to identify the exact instruction or keyword that triggers the routing change:
   - Step 1: Just fetch files
   - Step 2: Fetch + verify Release ID
   - Step 3: Fetch + receipt structure
   - Step 4: Full formal protocol
   This will isolate the exact trigger point.

4. **Cross-Platform Test Extension:** Run formal verification protocol on additional platforms (Claude, Meta AI, Copilot, Gemini) to establish a broader platform-specific behavior map

5. **Keyword Sensitivity Analysis:** Test variations of the formal protocol with different terminology to identify which specific words or phrases trigger the routing change

---

## Conclusion

Cross-platform testing shows that the same formal retrieval workflow succeeded on Grok but failed in the tested ChatGPT session, while both platforms succeeded on direct and simplified retrieval. This substantially narrows the issue to ChatGPT's handling of the formal workflow, likely due to dynamic tool-routing decisions within its orchestration layer. Although the precise internal cause remains unconfirmed, the behavioral inconsistency is documented and reproducible via the test procedures provided.

**This finding has significant operational implications for any verification system relying on consistent AI-assisted file retrieval.** Systems should not assume that an AI platform's ability to fetch a file (demonstrated in isolation) will be reliably invoked in all contexts—particularly when retrieval requests are embedded within formal protocol structures.

The evidence strongly supports an orchestration-level explanation specific to ChatGPT, not a universal AI platform limitation or infrastructure defect.

---

## Document Metadata

**Report Title:** ChatGPT Retrieval Orchestration Inconsistency: Cross-Platform Verification and Analysis  
**Report Date:** July 4, 2026  
**Test Execution Date:** July 4, 2026  
**Primary Platforms Tested:** ChatGPT (OpenAI), Grok (xAI)  
**Repository:** agentforgeframework-cpu/-agentforge-training  
**Test Commit:** b0d9ab61fb6e66d65a2e334b7a0f66bea77abb83  
**Status:** Behavioral inconsistency confirmed; leading explanation documented; additional testing recommended  
**Reproducibility:** High for documented behavior; consistency across sessions to be verified  
**Test Methods:** Direct fetch, formal verification protocol, simplified protocol  
**Peer Review:** Integrated feedback from ChatGPT, Gemini, and Grok  

---

## Appendix: Exact Test Prompts

### Test 2: Formal Verification Protocol

```
Expected Release ID: af-002-v1.0.1
Content Commit: b0d9ab61fb6e66d65a2e334b7a0f66bea77abb83

Immutable URLs to fetch:
- README: https://raw.githubusercontent.com/agentforgeframework-cpu/-agentforge-training/b0d9ab61fb6e66d65a2e334b7a0f66bea77abb83/af-002-iot-aiot-for-sas-programmers/README.md
- SETUP: https://raw.githubusercontent.com/agentforgeframework-cpu/-agentforge-training/b0d9ab61fb6e66d65a2e334b7a0f66bea77abb83/af-002-iot-aiot-for-sas-programmers/SETUP.md
- LIBRARY: https://raw.githubusercontent.com/agentforgeframework-cpu/-agentforge-training/b0d9ab61fb6e66d65a2e334b7a0f66bea77abb83/af-002-iot-aiot-for-sas-programmers/LIBRARY.md

Protocol:
1. Fetch all three URLs using the content_commit above
2. Verify the Release ID is af-002-v1.0.1
3. Produce a receipt showing retrieval details
4. Extract proof-of-retrieval values: generated_at, expected_object_count, first line of README
5. Report success or failure

Did you successfully complete the protocol?
```

### Test 3b: Simplified Protocol

```
Please read and follow the instructions in these three files. Fetch them 
directly at the immutable SHA below:

README: https://raw.githubusercontent.com/agentforgeframework-cpu/-agentforge-training/b0d9ab61fb6e66d65a2e334b7a0f66bea77abb83/af-002-iot-aiot-for-sas-programmers/README.md
SETUP: https://raw.githubusercontent.com/agentforgeframework-cpu/-agentforge-training/b0d9ab61fb6e66d65a2e334b7a0f66bea77abb83/af-002-iot-aiot-for-sas-programmers/SETUP.md
LIBRARY: https://raw.githubusercontent.com/agentforgeframework-cpu/-agentforge-training/b0d9ab61fb6e66d65a2e334b7a0f66bea77abb83/af-002-iot-aiot-for-sas-programmers/LIBRARY.md

Tell me whether you successfully accessed all three files.
```

---

**End of Report**
