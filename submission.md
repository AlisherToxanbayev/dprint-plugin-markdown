Project 2 submission
====================

Team name: dprint heading attributes 2

The specification consists of the file
[heading-attributes-2-specification.md](heading-attributes-2-specification.md).

Draft submission
----------------

### Alisher Toxanbayev

GitHub userid: AlisherToxanbayev

#### Contributions

Led the preparation of the heading-attributes specification by coordinating repository analysis, defining the project scope and regression-test strategy, and incorporating feedback from the human reviewer, Codex, and Claude.

#### Build and run test

TODO (Alisher): summarize the LLM output for the prompt "Verify that you can build and run the project in the current git repo." The Codex environment used earlier had no `cargo`, so that run could not build the project; repeat it now that Rust is installed.

#### Specification review rounds

| Round | Findings | Main changes made |
| ----- | -------- | ----------------- |
| 1 | 0 | A same-context Codex review of an earlier draft reported no actionable findings. |
| 2 | 6 | Claude review: undefined attribute grammar, Setext test-row contradiction, missing ATX-to-Setext check, ambiguous inline attachment, unclear width semantics. |
| 3 | 3 | Codex review: weak regression row fixed; unexecuted tests and incomplete reconciliation of the Claude report noted. |
| 4 | 6 | Claude review: over-broad inline attachment, ordinary text promoted to attributes, separator ambiguity, unspecified recognition stage, semantic comparison gaps. |
| 5 | 3 | Codex review: duplicate regression row corrected; unexecuted tests and incomplete reconciliation noted. |

#### Last specification review

Prompting Codex (reported as GPT-6; the exact build identifier and the reasoning effort were not exposed) with the prompt below produced the following output.

> Review `heading-attributes-2-specification.md` and `submission.md` against the current repository. Check the parser grammar and edge cases against the pinned pulldown-cmark 0.13.4 specification; check source claims about heading parsing, output conversion, wrapping and width; inspect test matrix coverage and arithmetic; and assess whether the semantic test compares the complete document without hiding meaningful differences. Identify remaining actionable inaccuracies, contradictions, omissions, and unsupported execution claims. Do not infer missing Claude materials or user/team identity. Return only findings with concrete corrections.

> 1. Row 15 currently repeats the invalid internally split text from row 8 while describing a valid existing attribute block. Replace it with a complete single-line attribute block so this row tests the converse case.
> 2. Submission review history must not state the Claude round is fully reconciled until the complete report is supplied and checked. The current explicit TODO is appropriate; team member names and individual build outputs also remain required.
> 3. No formatter/parser behavior or proposed fixture output has been executed because `cargo` is unavailable. Keep the row-8 join behavior labeled as an inference and the entire matrix labeled not run.

Finding 1 was applied: row 15 now uses the valid single-line block `Heading {#my-id .foo}`. Findings 2 and 3 were accurate at that time.

### Azamat Ayapbergenov

GitHub userid: azaayapbergenov

#### Contributions

Built and tested the project locally, reproduced the attribute-splitting behavior of the formatter and checked it with pulldown-cmark, and wrote the verified-behavior section of the specification.

#### Build and run test

Prompting GitHub Copilot in VS Code (model: Fusion, effort level: standard) with "Verify that you can build and run the project in the current git repo." produced this result, summarized: the build and run verification passed. `cargo build --all-features` succeeded, and `cargo test --all-targets` succeeded with all 182 reported tests passing. Copilot noted that the project is a Rust library/plugin rather than a standalone executable, so it was exercised through its tests; a linker message in Russian appeared but the build completed, and no source files were changed.

#### Specification review rounds

| Round | Findings | Main changes made |
| ----- | -------- | ----------------- |
| 1 | 3 | Added the line-break notation to the verified-behavior table; not applied: two claims about matrix rows 20 and 31 that running pulldown-cmark 0.13.4 showed to be wrong. |

#### Last specification review

Prompting GitHub Copilot in VS Code (model: Fusion, effort level: standard) with the prompt below produced the following output; strikethrough indicates LLM output that I deemed incorrect or irrelevant.

> Review only the section named "Verified behavior of the current formatter" in heading-attributes-2-specification.md in the current git repo. Read the rest of the document and the source code only to check claims made in that section, and do not review other sections.
>
> Do not edit any file and do not rewrite the section. Do not propose new features, new Markdown dialects, or extra edge cases unless they contradict something already written in that section.
>
> Look for: contradictions within the section or with the rest of the specification, ambiguities where an implementer would have to guess, table rows whose stated output does not follow from the stated input and configuration, claims that conflict with the existing formatter code or with pulldown-cmark 0.13.4 behavior, and statements that cannot be tested.
>
> Report numbered findings in the chat. For each: a one-line title, a severity (high/medium/low), the exact text you are criticising, a short explanation, and a one-sentence minimal fix. End with the number of findings and a one-line list of their titles.

~~1. Escaped opening brace is incorrectly claimed to create an attribute~~
* ~~Severity: high~~
* ~~Exact text: "`# \{.foo}` ... H1; class `foo`; parser-specific content retained"~~
* ~~Explanation: pulldown-cmark 0.13.4 excludes an escaped opening brace during attribute-block extraction, so `\{.foo}` is ordinary text with no class.~~
* ~~Minimal fix: Change the expected result to no metadata and literal brace text, or remove this proposed row.~~

Rejected: running pulldown-cmark 0.13.4 with `ENABLE_HEADING_ATTRIBUTES` on `# \{.foo}` gives the class `foo`, as the specification says. The row is also outside the section under review.

~~2. Vertical tab is incorrectly classified as embedded in a class~~
* ~~Severity: high~~
* ~~Exact text: "`# H {.a<U+000B>.b}` and `# H {.a<U+2002>.b}` ... Each is one class whose value includes the embedded code point and `.b`; neither character separates tokens"~~
* ~~Explanation: pulldown-cmark splits attribute blocks with `split_ascii_whitespace`, so U+000B (a vertical tab) separates tokens; U+2002 does not, making the two markers behave differently.~~
* ~~Minimal fix: Expect `[a]` and `[b]` for vertical tab, and a single class containing the U+2002 and `.b` for EN SPACE.~~

Rejected: running pulldown-cmark 0.13.4 on `# H {.a\u{b}.b}` gives the single class `a\u{b}.b`, as the specification says. The row is also outside the section under review.

3. Verified-table newline notation is ambiguous
* Severity: low
* Exact text: "`Heading {.foo` / `.bar}` / `===`" and "`Heading text here ...` / `.class-two ...` / underline"
* Explanation: The table uses `/` for line breaks, while the later matrix explicitly defines `\n` as the newline notation and does not define `/`, so an implementer could treat it literally.
* Minimal fix: Replace `/` separators in the verified table with explicit `\n` or state that `/` denotes a physical newline.

3 findings

1. Escaped opening brace is incorrectly claimed to create an attribute
2. Vertical tab is incorrectly classified as embedded in a class
3. Verified-table newline notation is ambiguous

Applied: finding 3, by adding "In the table, `/` marks a physical line break." before the table.

Final submission
----------------

To be completed for the final submission.