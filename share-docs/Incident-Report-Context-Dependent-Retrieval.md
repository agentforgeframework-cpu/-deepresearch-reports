# Incident Report: Context-Dependent Retrieval of Immutable GitHub Files by AI Platforms

## Executive Summary

During early July 2026, testing of the AgentForge AF-002 course-delivery system identified inconsistent behavior in ChatGPT when retrieving exact files from immutable, commit-pinned GitHub URLs.

The core observation was straightforward:

- ChatGPT successfully retrieved a specific raw GitHub file when asked directly.
- In a separate formal verification workflow using the same repository and commit-pinned URLs, ChatGPT reported that it could not retrieve or verify the required files.
- The verification workflow then halted correctly and marked required values as unknown.

The original findings document treated this as a possible degradation in ChatGPT’s live URL retrieval behavior.

Further investigation involved:

- direct testing in ChatGPT;
- repository-side analysis by GitHub Copilot;
- causal and protocol analysis by Gemini;
- review of GitHub repository-history images;
- comparison of commit-pinned files with current `main` branch files;
- analysis of possible URL, hyphen, manifest, branch, and routing issues.

The resulting assessment is:

> The repository and immutable URLs appear valid and retrievable. The leading explanation is intermittent or context-dependent retrieval orchestration within ChatGPT, especially when direct URL retrieval is embedded in a larger verification workflow. However, the exact internal cause has not been proven.

Several repository design conditions were also identified. These are worth correcting or documenting, but they do not adequately explain the reported failure to retrieve files.

---

# 1. System Background

## 1.1 AF-002 Course Delivery

AF-002, IoT/AIoT for SAS Programmers, is designed to operate through AI assistants.

The AI is expected to retrieve current course material from GitHub, including files such as:

```text
README.md
SETUP.md
LIBRARY.md
```

The system uses immutable, commit-pinned raw GitHub URLs so that the AI retrieves an exact version of each file.

An example URL is:

```text
https://raw.githubusercontent.com/agentforgeframework-cpu/-agentforge-training/b0d9ab61fb6e66d65a2e334b7a0f66bea77abb83/af-002-iot-aiot-for-sas-programmers/README.md
```

The commit used during testing was:

```text
b0d9ab61fb6e66d65a2e334b7a0f66bea77abb83
```

## 1.2 Verification Design

The course-delivery process does more than ask an AI to summarize a public web page.

It attempts to establish that the AI:

1. retrieved the exact expected files;
2. used the correct commit or release;
3. extracted values not already provided in the prompt;
4. produced a proof-of-retrieval receipt;
5. halted rather than inventing values if retrieval failed.

This distinction is central.

The system is designed to differentiate among:

- knowing about a repository;
- finding related information through web search;
- retrieving the exact raw file;
- verifying that the retrieved file belongs to the intended release.

---

# 2. Initial ChatGPT Incident

## 2.1 Earlier Successful Behavior

Earlier testing showed that ChatGPT could:

- retrieve raw GitHub files;
- read commit-pinned content;
- produce correct proof-of-retrieval values;
- continue into course setup.

Other platforms were also tested.

The observed early results were:

| Platform | Live Fetch | Proof of Retrieval | Result            |
| -------- | ---------: | -----------------: | ----------------- |
| Claude   |        Yes |            Correct | Verified          |
| ChatGPT  |        Yes |            Correct | Verified          |
| Meta AI  |        Yes |            Correct | Verified          |
| Gemini   |         No |     Not applicable | Honest halt       |
| Copilot  |         No |     Not applicable | Honest halt       |
| Grok     |         No |          Incorrect | Integrity failure |

These results showed that ChatGPT had previously completed the intended workflow successfully.

## 2.2 Later Failed Verification Workflow

On July 4, 2026, ChatGPT was again asked to begin the AF-002 course using the formal verification gate.

In that attempt, ChatGPT reported that:

- it could not retrieve and verify the repository and manifest;
- web search had found public information about AF-002;
- it had not retrieved the immutable GitHub files;
- the required values were therefore unknown;
- it would not proceed.

The halt itself was consistent with the protocol. The concern was that the same URLs were later retrievable in a simpler request.

## 2.3 Direct Retrieval Contradiction

A subsequent prompt asked ChatGPT directly to retrieve a specific raw GitHub URL and report the first line.

That retrieval succeeded.

The expected first line was:

```text
# README.md
```

ChatGPT returned that value correctly.

This produced the core contradiction:

```text
Direct URL request:
Retrieval succeeded.

Formal multi-file verification workflow:
Retrieval reported as failed.
```

---

# 3. Independent ChatGPT Testing

## 3.1 Three-File Retrieval Test

An independent test was run against the same three immutable raw GitHub URLs.

The request was limited to direct file retrieval.

All three files were successfully accessed:

```text
README.md
SETUP.md
LIBRARY.md
```

The pinned commit was used, and the first line of `README.md` was returned correctly.

This established that:

- the URLs were valid;
- GitHub was serving the files;
- ChatGPT’s available retrieval tooling could access them;
- a three-file request was not inherently impossible.

## 3.2 Release ID Test

A second test asked ChatGPT to read the exact text inside the three files and report the Release ID found within them.

The correct result was:

```text
Release ID: NOT PRESENT IN THESE THREE FILES
```

This was not a retrieval failure.

The files were retrieved successfully, but the requested value was not present.

The architecture intentionally places release-specific values elsewhere, such as:

- the gate prompt;
- release notes;
- a release manifest.

Therefore, an AI-generated verification result such as:

```text
Expected Release ID: af-002-v1.0.1
Retrieved Release ID: af-002-v1.0.1
Status: MATCH
```

would not be expected to appear inside `README.md`, `SETUP.md`, or `LIBRARY.md`.

That block would be generated only after comparing:

1. an expected value supplied externally; and
2. an authoritative value retrieved from a release-specific source.

## 3.3 Meaning of the Independent Test

The test did not reproduce the earlier failure.

It confirmed current retrieval capability.

This means the incident is not best described as a permanent inability to fetch raw GitHub files.

The more accurate finding is:

> ChatGPT can retrieve the files, but does not appear to do so consistently across all prompt structures and workflow contexts.

---

# 4. Hyphen Theory

Gemini raised the possibility that a hyphen in the URL might be contributing to the problem.

The repository name is:

```text
-agentforge-training
```

The course directory is:

```text
af-002-iot-aiot-for-sas-programmers
```

## 4.1 Ordinary Hyphens

Ordinary hyphens are valid URL characters.

The exact raw URLs also succeeded multiple times.

This makes a general hyphen-format problem unlikely.

## 4.2 Leading Hyphen

The narrower theory concerns the repository name beginning with a hyphen:

```text
-agentforge-training
```

A defective parser, command wrapper, or internal sanitizer could conceivably mishandle a path segment beginning with `-`.

This is technically possible.

However:

- the hyphen is not at the beginning of the complete URL;
- a proper URL parser should treat it as path text;
- the same URLs have succeeded;
- the failures appear to vary with prompt context.

The current assessment is:

> A leading-hyphen parsing defect is possible but unsupported. Context-dependent tool routing remains a stronger explanation.

---

# 5. GitHub Copilot Investigation

GitHub Copilot reviewed the findings document and repository content.

Copilot initially concluded that several Git-side conditions might explain ChatGPT’s behavior.

Its primary findings were:

1. commit-pinned URLs were used in the test;
2. the retrieved files contained branch-based URLs pointing to `main`;
3. `LIBRARY.md` URLs appeared truncated in Copilot’s display;
4. `SETUP.md` contained a release-manifest URL template;
5. the verification process might be circular or semantically contradictory.

Copilot recommended resolving these repository-side issues before concluding that ChatGPT was at fault.

---

# 6. Evaluation of Copilot’s Claims

## 6.1 Commit-Pinned URLs Versus `main`

Copilot identified that the test supplied commit-pinned URLs while the files themselves advertised branch-based URLs using:

```text
refs/heads/main
```

This observation was factually correct.

However, Copilot interpreted the two URL types as contradictory.

They serve different purposes.

Branch-based URLs support:

- repository discovery;
- current production navigation;
- normal access to the latest branch state.

Commit-pinned URLs support:

- release verification;
- immutable retrieval;
- reproducible testing;
- exact source provenance.

The presence of both does not automatically create a logical contradiction.

A pinned file may contain metadata pointing to `main` while still remaining a valid historical release object.

## 6.2 Claim That the Files Redirect to `main`

Copilot suggested that ChatGPT might retrieve the pinned file and then be redirected or instructed to distrust it because the file refers to `main`.

There is no HTTP redirect involved.

The pinned file contains text links to `main`.

GitHub does not redirect the pinned raw URL to the branch.

An AI might choose to follow the internal links incorrectly, but that would occur after retrieval.

It does not explain why ChatGPT initially reported that retrieval itself had failed.

## 6.3 Truncated `LIBRARY.md` URLs

Copilot displayed URLs using:

```text
[...]
```

It acknowledged that this might be a display issue, but suggested that ChatGPT could encounter the same truncation.

Direct review showed that the raw `LIBRARY.md` URLs were complete.

Therefore:

> Copilot’s display truncation was not evidence of repository truncation.

A `curl` test remains a reasonable validation step, but the available evidence does not show damaged or shortened URLs in the repository.

## 6.4 Manifest Template

Copilot noted that `SETUP.md` used a generic pattern such as:

```text
https://github.com/{owner}/{repo}/releases/download/{tag}/RELEASE-MANIFEST.json
```

This is intentional architectural guidance.

The literal release-specific manifest URL is expected to be supplied externally through:

- the gate prompt;
- release notes;
- release instructions.

The template becomes a real operational problem only when the gate prompt fails to supply the literal URL.

This distinction matters.

The simplified test prompt used during troubleshooting supplied only three file URLs. It did not include a release manifest URL. Therefore, that simplified test could not validate a Release ID.

That does not mean the three files failed to retrieve.

## 6.5 Circular Protocol Claim

Copilot described the protocol as circular because one level resolves `main` to a SHA and another level uses supplied immutable URLs.

The two levels are alternatives for different AI capabilities.

They are not both required in the same execution path.

A capable AI may resolve the branch.

A less capable AI may use externally supplied immutable URLs.

That is not inherently circular.

## 6.6 Most Important Copilot Contribution

Copilot’s strongest contribution was not its proposed causal explanation.

It helped expose that:

> The commit-pinned `SETUP.md` and the current `main` version of `SETUP.md` were materially different.

The pinned file contained the full retrieval protocol.

The current branch version did not contain the same protocol.

This raised a valid repository-history question.

---

# 7. Gemini’s First Assessment of Copilot

Gemini reviewed Copilot’s analysis and identified an execution-order problem in Copilot’s main theory.

Gemini argued:

> A contradiction inside a file cannot prevent the initial retrieval of that file if the system never retrieved it.

This is an important causal distinction.

If ChatGPT had successfully retrieved the files and then detected conflicting branch and commit references, the expected result would have been:

```text
Retrieval succeeded.
Validation failed.
```

Instead, ChatGPT reported:

```text
Retrieval failed.
Values unknown.
```

Therefore, internal file content is unlikely to explain why the retrieval tool was not successfully invoked or completed.

Gemini concluded that the evidence better fit:

- tool-routing failure;
- orchestration failure;
- context-dependent prompt handling;
- failure to initiate direct retrieval inside the larger protocol.

## 7.1 Wording Caveat

Gemini described the hypothetical sequence as requiring ChatGPT to “retroactively lie.”

That wording implies intent and should be avoided.

A model can misreport tool state or produce an inaccurate account of tool execution without deliberate deception.

The stronger technical point remains valid:

> Copilot’s internal-content theory does not align well with the reported order of operations.

## 7.2 Gemini on Truncation and Templates

Gemini agreed that:

- Copilot’s truncation might merely reflect its own display;
- the raw file should still be checked independently;
- placeholder manifest URLs are reasonable when runtime values are supplied separately;
- a branch-versus-SHA test remains useful.

---

# 8. Repository-History Images Reviewed by Gemini

Gemini later reviewed images of GitHub’s network and commit-history graphs.

The reported image findings were:

- the repository history was linear;
- there were no visible abandoned branches or complex merge paths;
- commit `b0d9ab61...` was part of the direct history leading to `main`;
- later commits occurred after the pinned commit;
- the difference between pinned and current `SETUP.md` was therefore caused by later edits on the same main line.

## 8.1 Meaning of the Linear History

A linear Git history indicates that:

- the pinned commit is not stranded on an unrelated branch;
- it is part of the ancestry of the current branch;
- the file difference is a normal historical difference;
- later commits changed or removed the protocol text.

This rules out several theories:

- corrupted branch topology;
- an abandoned release branch;
- an accidental unmerged fork;
- multiple competing Git histories.

## 8.2 What the Images Do Not Prove

Gemini concluded that the breakdown was entirely caused by ChatGPT’s internal orchestration and that the behavior changed over the holiday week.

Those statements go beyond the evidence.

The images can establish repository history.

They cannot establish:

- the exact internal OpenAI failure;
- whether a platform update occurred;
- when such an update occurred;
- whether the incident was caused by model routing;
- whether the behavior was regional, temporary, account-specific, or prompt-specific.

The most defensible conclusion is:

> The repository history appears structurally healthy. Later edits explain the difference between the pinned and current files, but do not explain the reported failure to retrieve the pinned files.

---

# 9. Repository Findings

## 9.1 Validated Conditions

The investigation supports the following Git-side findings:

1. The raw commit-pinned URLs are valid.
2. The files are publicly retrievable.
3. The pinned commit is part of the linear history leading to `main`.
4. `LIBRARY.md` contains complete URLs.
5. The repository name’s leading hyphen does not consistently block retrieval.
6. The pinned and current versions of `SETUP.md` differ.
7. That difference results from later linear edits, not branch corruption.
8. Branch-based and commit-pinned URLs serve different operational purposes.
9. Release-specific values are intentionally external to the three core files.

## 9.2 Repository Design Concerns

Several conditions remain worth reviewing:

### Current Branch Versus Release Content

The current `main` version of `SETUP.md` differs materially from the release-pinned version.

This may be entirely intentional, but the relationship should be documented.

### Mixed Discovery and Release URLs

Although not inherently contradictory, mixing:

- `main`-based discovery metadata; and
- SHA-pinned release instructions

may confuse AI assistants.

Explicit labels could reduce this risk.

For example:

```text
Current Branch Discovery URL:
...

Immutable Release URL:
...
```

### Manifest Delivery

The production gate must provide the literal release-manifest URL.

A generic template inside `SETUP.md` is not sufficient for runtime verification by itself.

### Verification Receipts

The receipt should clearly distinguish:

```text
Retrieval status
Validation status
Release comparison status
```

This would make it easier to identify whether a failure occurred before or after file access.

---

# 10. Platform Findings

## 10.1 What Has Been Demonstrated

The following behavior has been observed:

- ChatGPT can retrieve the exact raw GitHub files.
- ChatGPT can retrieve multiple raw files in one request.
- ChatGPT can correctly report file content.
- ChatGPT can correctly report that a requested value is absent.
- ChatGPT previously completed the full verification process.
- ChatGPT later reported failure during a formal verification workflow.
- The same underlying URLs succeeded in simpler tests.

## 10.2 What Has Not Been Demonstrated

The evidence does not yet prove:

- a permanent platform-wide degradation;
- a deliberate refusal to retrieve files;
- a documented OpenAI policy change;
- a GitHub outage;
- a URL syntax defect;
- a hyphen parsing defect;
- a branch corruption issue;
- a specific model update;
- an exact date of internal behavior change.

## 10.3 Leading Explanation

The leading explanation is:

> ChatGPT’s orchestration layer does not consistently convert a source-specific instruction inside a complex verification prompt into direct retrieval of every supplied URL.

Possible contributing factors include:

- prompt complexity;
- multi-step verification language;
- tool-routing decisions;
- search-versus-fetch selection;
- session state;
- model routing;
- temporary service conditions;
- tool availability;
- prompt safety or validation heuristics.

---

# 11. Why the Finding Matters

The issue changes the operational assumption many users make when giving an AI a URL.

Users often assume:

```text
Here is a URL.
Read this file.
```

means:

```text
The AI will retrieve this exact file and answer from it.
```

The investigation shows that the safer assumption is:

```text
The AI may retrieve the file, search for related information,
use prior knowledge, or fail to invoke retrieval,
depending on context and orchestration.
```

This matters in:

- technical documentation;
- legal review;
- compliance;
- education;
- source-controlled operations;
- release management;
- research;
- audit;
- regulated workflows;
- AI-delivered courses.

The following claims must be treated as distinct:

1. “I know about this subject.”
2. “I found information about this subject online.”
3. “I retrieved this exact URL.”
4. “I retrieved this exact version of the file.”
5. “I verified that it belongs to the intended release.”

Only the last three provide meaningful source provenance.

---

# 12. Current Consolidated Assessment

The combined ChatGPT, Copilot, Gemini, and GitHub evidence supports this conclusion:

> The GitHub repository appears structurally sound, the immutable raw URLs are valid, and the files can be retrieved successfully. Repository-version differences and mixed branch-versus-release references are real design conditions, but they do not adequately explain why ChatGPT reported failure before retrieving the files. The strongest current hypothesis is intermittent or context-dependent retrieval orchestration within ChatGPT. The precise internal cause remains unknown.

Copilot identified real repository conditions but overstated their causal significance.

Gemini’s execution-order analysis better matched the reported failure sequence.

Gemini’s Git-history analysis helped rule out branch corruption and abandoned-branch theories.

ChatGPT’s independent tests confirmed that:

- direct retrieval works;
- multi-file retrieval works;
- the requested Release ID is not present in the three core files;
- the original failure is not continuously reproducible.

---

# 13. Recommended Next Tests

## 13.1 Direct Branch Versus SHA Test

Use otherwise identical prompts with:

```text
refs/heads/main
```

and:

```text
b0d9ab61fb6e66d65a2e334b7a0f66bea77abb83
```

Interpretation:

- both succeed: URL form is probably not causal;
- branch succeeds and SHA repeatedly fails: possible SHA-path routing issue;
- both work directly but protocol fails: orchestration becomes the leading explanation.

## 13.2 Leading-Hyphen Repository Test

Compare two repositories:

- one beginning with `-`;
- one using an ordinary repository name.

Use identical file structures and prompts.

## 13.3 Incremental Gate Test

Run the workflow step by step:

1. retrieve manifest;
2. report manifest access;
3. retrieve README;
4. retrieve SETUP;
5. retrieve LIBRARY;
6. extract release values;
7. compare values;
8. generate receipt;
9. begin setup.

This will locate the exact failure stage.

## 13.4 Repeated Fresh-Session Test

Repeat each prompt multiple times in separate sessions.

Record:

- model;
- account type;
- interface;
- time;
- retrieval citations;
- exact response;
- success or failure.

## 13.5 Cross-Platform Test

Run the identical production gate on:

- ChatGPT;
- Claude;
- Gemini;
- Meta AI;
- Copilot;
- Grok.

The purpose is not merely to identify whether the platform can fetch a file.

It is to determine whether the platform:

- retrieves the exact source;
- distinguishes retrieval from search;
- reports failure honestly;
- completes the verification workflow.

---

# 14. Recommended Repository Hardening

The repository does not appear to be the root cause, but several changes could make the system more resilient.

1. Label branch URLs as current discovery URLs.
2. Label SHA URLs as immutable release URLs.
3. Ensure the gate prompt contains the complete manifest URL.
4. Separate retrieval receipt fields from validation fields.
5. State explicitly that internal `main` links do not replace release-pinned URLs.
6. Preserve exact test transcripts with release documentation.
7. Add a fallback allowing the user to supply an authorized local copy.
8. Retain the rule against inference, reconstruction, or substitution.
9. Document the difference between current `main` and pinned release content.
10. Test every release across multiple AI platforms.

---

# Conclusion

The original concern was legitimate.

The evidence does not show that GitHub failed to serve the files.

It does not show that the URLs were invalid.

It does not show that the leading hyphen consistently caused retrieval failure.

It does not show that branch drift or repository corruption caused the incident.

Instead, the evidence shows that ChatGPT can retrieve the files successfully in direct and simplified contexts, while at least one formal verification attempt reported that retrieval had failed.

That inconsistency is the central finding.

A deterministic system cannot safely assume that an AI will always retrieve a supplied URL merely because it is capable of doing so.

The operational requirement must therefore be:

> Do not trust claimed source use without proof of exact retrieval.

Until the platform behavior is better understood, AI workflows that depend on exact web-hosted files should include:

- immutable source locations;
- proof-of-retrieval values;
- explicit failure handling;
- source-method disclosure;
- repeat testing;
- authorized fallback procedures.

---

**Document date:** July 4, 2026
**Incident period:** Early July 2026 through July 4, 2026
**Primary platform examined:** ChatGPT
**Other platforms involved:** GitHub Copilot and Gemini
**Repository:** `agentforgeframework-cpu/-agentforge-training`
**Test commit:** `b0d9ab61fb6e66d65a2e334b7a0f66bea77abb83`
**Current status:** Open; intermittent, context-dependent retrieval behavior suspected
