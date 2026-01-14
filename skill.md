---
name: wechat-article-formatter
description: Format WeChat article markdown files into beautifully styled HTML using bm.md service. Supports custom styling optimized for WeChat public account articles.
---

# WeChat Article Formatter Skill

This skill formats markdown files into styled HTML optimized for WeChat public account articles using the bm.md rendering service.

## Overview

The skill performs the following workflow:
1. Reads the provided markdown file or content
2. Loads custom CSS styling from the `styles/custom.css` file in this skill's directory
3. Calls the bm.md API to render the markdown with WeChat-optimized styling
4. Outputs the formatted HTML that can be directly pasted into WeChat's article editor

## Usage

When the user invokes this skill, you should:

### Step 1: Identify the Input

The user will provide one of the following:
- A file path to a markdown file (e.g., `/path/to/article.md`)
- Raw markdown content directly in the conversation

If the input is unclear, ask the user to provide either a markdown file path or paste the markdown content directly.

### Step 2: Read the Custom CSS

Read the custom CSS file from this skill's directory:

```
{{SKILL_DIR}}/styles/custom.css
```

Use the Read tool to get the CSS content. This CSS provides the custom styling for the WeChat article.

### Step 3: Call the bm.md API

Make a POST request to the bm.md rendering API with the following configuration:

**Endpoint:** `POST https://bm.md/api/markdown/render`

**Request Body:**
```json
{
  "markdown": "<the markdown content>",
  "codeTheme": "GreenSimple",
  "platform": "wechat",
  "enableFootnoteLinks": true,
  "openLinksInNewWindow": true,
  "customCss": "<content from styles/custom.css>"
}
```

Use the Bash tool with curl to make the API request:

```bash
curl -X POST https://bm.md/api/markdown/render \
  -H "Content-Type: application/json" \
  -d '{
    "markdown": "<escaped markdown content>",
    "codeTheme": "GreenSimple",
    "platform": "wechat",
    "enableFootnoteLinks": true,
    "openLinksInNewWindow": true,
    "customCss": "<escaped CSS content>"
  }'
```

**Important Notes:**
- Properly escape the markdown and CSS content for JSON (escape quotes, newlines, etc.)
- The response will be JSON with the rendered HTML in the `result` field

### Step 4: Output the Result

After receiving the API response:

1. Extract the HTML from the `result` field in the JSON response
2. Save the HTML to a file (default: same name as input with `.html` extension, or `output.html` if content was provided directly)
3. Display a success message with the output file path

### Step 5: Offer to Publish (Optional)

After formatting is complete, ask the user if they want to publish the article to WeChat:

> "The article has been formatted and saved to `<output_path>`. Would you like me to publish it as a draft to your WeChat public account using the `/wechat-article-publisher` skill?"

If the user confirms, invoke the `/wechat-article-publisher` skill with the formatted HTML content.

## Example Workflow

**User:** Format my article at `/Users/james/articles/my-post.md`

**Assistant Actions:**
1. Read the markdown file at `/Users/james/articles/my-post.md`
2. Read the custom CSS from this skill's `styles/custom.css`
3. Call the bm.md API with the markdown and CSS
4. Save the result to `/Users/james/articles/my-post.html`
5. Report success and ask about publishing

## API Response Format

The bm.md API returns:
```json
{
  "result": "<div id=\"bm-md\">...</div>"
}
```

The `result` field contains the fully styled HTML with inline CSS, ready to be copied into WeChat's rich text editor.

## Styling Features

The custom CSS provides:
- Clean typography with Optima/Microsoft YaHei fonts
- Green accent color theme (rgb(53, 179, 120))
- Properly styled headings, paragraphs, and lists
- Code blocks with syntax highlighting
- Blockquotes with left border accent
- Responsive tables
- Optimized spacing for mobile reading

## Error Handling

If the API call fails:
1. Display the error message to the user
2. Suggest checking the markdown content for any issues
3. Offer to retry the request

If the markdown file cannot be read:
1. Verify the file path exists
2. Check file permissions
3. Report the specific error to the user

## Dependencies

This skill can integrate with:
- `/wechat-article-publisher` - For publishing formatted articles as drafts to WeChat public accounts
