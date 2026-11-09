# Heading Attributes Preservation During Markdown Formatting

## Goal

The goal of this project is to ensure that Markdown heading attributes remain correctly associated with their headings when the `dprint-plugin-markdown` formatter wraps long lines.

The formatter automatically wraps long Markdown headings according to the configured line width. However, wrapping a heading that contains an attribute may introduce a newline between the heading content and its attribute.

For example, consider the following input:

```markdown
# This is a very long heading that needs to be wrapped {#my-id}
```

An incorrect formatter output may look like this:

```markdown
# This is a very long heading that
needs to be wrapped
{#my-id}
```

In this case, the Markdown parser may no longer recognize `{#my-id}` as an attribute of the heading.

The required behavior is to preserve the association between the heading and its attribute:

```markdown
# This is a very long heading that
needs to be wrapped {#my-id}
```

The heading content may be wrapped across multiple lines, but its attribute must remain attached to the heading.

The primary requirement is that formatting must preserve the semantic interpretation of the original Markdown document.

## Scope

This specification covers the handling of heading attributes during automatic Markdown line wrapping.

The initial scope includes:

- ATX headings containing an ID attribute, such as `{#my-id}`.
- Long headings that require multiple output lines.
- Preservation of heading attributes during automatic wrapping.
- Verification of the resulting Markdown using `pulldown-cmark`.

The implementation should follow the existing formatter architecture and preserve its current behavior for Markdown constructs unrelated to heading attributes.

Support for additional attribute forms should follow the syntax already supported by the repository and must not be assumed without verification.

## Heading Attributes

A heading attribute is additional metadata associated with a Markdown heading.

For this project, the primary example is an ID attribute:

```markdown
# Heading text {#my-id}
```

The expected HTML representation is conceptually:

```html
<h1 id="my-id">Heading text</h1>
```

The attribute is part of the heading syntax and must not be interpreted as independent paragraph text.

The formatter must preserve:

- The association between the attribute and its heading.
- The attribute's identifier.
- The heading level.
- The heading's textual content.

## Formatting Requirements

### 1. Attribute Preservation

A heading attribute must remain associated with its original heading after formatting.

The formatter must not introduce a line break that separates the attribute from the heading content in a way that changes how the Markdown parser interprets it.

### 2. Word Wrapping

Long heading content may be split across multiple lines according to the configured text width.

The attribute must remain on the same physical line as the final portion of the heading content.

For example:

**Input:**

```markdown
# A long heading that requires wrapping before reaching the configured width {#example}
```

**Expected output:**

```markdown
# A long heading that requires wrapping
before reaching the configured width {#example}
```

The formatter must not move the attribute to a separate line.

### 3. Attribute Integrity

The formatter must not modify the attribute's identifier or introduce additional characters into its syntax.

For example:

```markdown
# Example heading {#section-1}
```

must preserve the identifier `section-1` after formatting.

The formatter must not:

- Remove the attribute.
- Change its identifier.
- Split the attribute across multiple lines.
- Move it into an independent paragraph.
- Convert it into ordinary heading text.

### 4. Semantic Preservation

The Markdown document before and after formatting must preserve the same heading semantics.

In particular, the following properties must remain unchanged:

- Heading level.
- Heading content.
- Heading attribute association.
- Heading ID in the resulting HTML.

Formatting is considered incorrect if it changes the resulting heading structure or removes its ID.

### 5. Existing Formatting Behavior

The implementation must preserve existing formatting behavior for headings without attributes.

It must also avoid introducing regressions in other Markdown elements.

The solution should be limited to the handling of heading attributes and should not unnecessarily change the general line-wrapping algorithm.

## Implementation Strategy

The implementation should follow these stages.

### Stage 1: Inspect Heading Processing

Examine the existing Markdown formatter to understand:

- How headings are identified.
- How heading content is represented.
- How heading attributes are recognized.
- Where automatic line wrapping is performed.
- How the formatter handles trailing heading syntax.

The implementation should reuse existing parsing and formatting structures whenever possible.

### Stage 2: Identify Heading Attributes

Determine how the existing parser represents heading attributes.

The formatter must be able to distinguish heading metadata from ordinary heading text.

An attribute must be treated as part of the heading structure rather than as ordinary text eligible for independent line wrapping.

### Stage 3: Adjust Line Wrapping

Modify the relevant formatting logic so that a heading attribute cannot become detached from its heading.

The heading text may be wrapped normally, but the attribute must remain attached to the final heading line.

When the configured line width cannot be satisfied without separating the attribute, preserving the heading's semantic correctness takes priority over the wrapping preference.

### Stage 4: Preserve Existing Behavior

Verify that headings without attributes continue to be formatted as before.

Ensure that the change does not introduce unintended modifications to paragraphs, lists, blockquotes, or other Markdown elements.

### Stage 5: Validate the Result

Use `pulldown-cmark` to convert the original and formatted Markdown into HTML.

Compare the resulting heading structure and attributes to verify that formatting preserves their meaning.

The parser configuration used for verification must support the heading attribute syntax recognized by the target project.

## Testing Requirements

Tests should cover the following scenarios.

### Basic Attribute Handling

**Input:**

```markdown
# Heading {#my-id}
```

Expected behavior:

- The heading retains its ID.
- Formatting does not remove or modify the attribute.

### Long Heading with an Attribute

**Input:**

```markdown
# This is a very long heading that requires automatic wrapping {#my-id}
```

Expected behavior:

- Heading content may be wrapped.
- The attribute remains attached to the final heading line.
- The resulting HTML contains the original heading ID.

### Attribute on a Separate Line

**Input:**

```markdown
# This is a long heading
that needs wrapping
{#my-id}
```

Expected behavior:

The formatter must not produce this structure if it causes the ID to stop being recognized as a heading attribute.

### Heading Without an Attribute

**Input:**

```markdown
# This is a very long heading without any attributes
```

Expected behavior:

Existing line-wrapping behavior must remain unchanged.

### Different Heading Levels

Test headings of different levels where heading attributes are supported by the existing parser.

The heading level and associated attribute must remain unchanged after formatting.

### Configured Line Width

Test headings using a relatively small configured line width, including a width of 40 characters as demonstrated during the project meeting.

Verify that the attribute remains attached even when preserving it causes the final line to exceed the preferred width.

## Semantic Equivalence Testing

The primary validation method is to compare the Markdown interpretation before and after formatting.

For every relevant fixture:

1. Provide the original Markdown input.
2. Convert the original input to HTML using the configured `pulldown-cmark` parser.
3. Format the Markdown using `dprint-plugin-markdown`.
4. Convert the formatted Markdown to HTML using the same parser configuration.
5. Compare the resulting heading semantics.

For example, both versions should produce an equivalent heading:

```html
<h1 id="my-id">This is a very long heading that needs wrapping</h1>
```

The comparison must verify that the heading ID has not been lost and that the heading content remains equivalent.

Tests must use the same parser version and relevant extension options for both conversions.

## Acceptance Criteria

The implementation is considered complete when:

1. Heading attributes remain associated with their original headings after formatting.
2. Long headings can be wrapped without detaching their attributes.
3. Heading IDs remain unchanged.
4. The resulting Markdown produces equivalent heading semantics before and after formatting.
5. Headings without attributes retain their existing formatting behavior.
6. Regression tests pass.
7. The solution does not introduce unrelated changes to the formatter.

## Non-Goals

The following tasks are outside the scope of this project:

- Implementing new Markdown heading attribute syntax.
- Changing how `pulldown-cmark` parses heading attributes.
- Modifying the general Markdown line-wrapping algorithm for unrelated elements.
- Implementing unrelated Markdown extensions.
- Redesigning heading formatting beyond what is required to preserve heading attributes.

## Reference

The expected behavior is based on the project meeting with Bryan Jones on November 8, 2026.

The demonstrated requirement is that a heading attribute must not be moved onto a separate line during word wrapping, because doing so can change the resulting HTML heading and cause its ID to be lost.