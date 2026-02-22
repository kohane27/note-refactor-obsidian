# Fix: Wiki-Style Link Generation in Frontmatter

## Problem

When using the "Extract selection to new note" feature with a template containing `{{link}}` placeholder, the generated frontmatter contained invalid markdown-style links:

```yaml
---
link: [01. Life Is Difficult](01.%20Life%20Is%20Difficult.md)
extracted: 2026-02-22T19:30
---
```

This format is not recognized as a valid Obsidian wiki-link.

## Expected Behavior

The link should be generated in Obsidian's wiki-link format:

```yaml
---
link: [[01. Life Is Difficult]]
extracted: 2026-02-22T19:30
---
```

## Root Cause

The plugin was using Obsidian's `generateMarkdownLink()` API which creates standard markdown links `[text](url)` instead of wiki-style links `[[text]]`.

Two locations were affected:

1. `src/doc.ts:markdownLink()` - Used for `{{new_note_link}}` placeholder
2. `src/main.ts` - Used for `{{link}}` placeholder (the link back to the original note)

## Fix Applied

### 1. src/doc.ts

Changed the `markdownLink` method to generate wiki-style links directly:

```typescript
async markdownLink(filePath: string){
  const file = await this.vault.getMarkdownFiles().filter(f => f.path === filePath)[0];
  if (!file) return '';
  return `[[${file.basename}]]`;
}
```

### 2. src/main.ts

Changed both occurrences (lines 154 and 175) to use the fixed `markdownLink` method:

```typescript
// Before:
const link = await this.app.fileManager.generateMarkdownLink(
  mdView.file,
  "",
  "",
  "",
);

// After:
const link = await this.NRDoc.markdownLink(mdView.file.path);
```

## Template Placeholders

After this fix, both link placeholders generate wiki-style links:

- `{{link}}` - Link to the original/source note (now `[[Note Name]]`)
- `{{new_note_link}}` - Link to the newly created note (now `[[Note Name]]`)

## Testing

1. Configure a template in plugin settings with `link: {{link}}`
2. Use "Extract selection to new note - first line as filename"
3. Verify the generated frontmatter contains `link: [[Note Name]]`
