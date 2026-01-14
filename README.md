# WeChat Article Formatter Skill

A Claude Code skill that formats markdown files into beautifully styled HTML optimized for WeChat public account articles.

## Features

- Converts Markdown to styled HTML using [bm.md](https://bm.md) rendering service
- Custom CSS styling optimized for WeChat article readability
- Green accent color theme with clean typography
- Support for GFM (GitHub Flavored Markdown) syntax
- Code syntax highlighting with GreenSimple theme
- Automatic footnote link conversion
- Optional integration with WeChat article publisher

## Installation

Add this skill to your Claude Code installation:

```bash
claude skill add git@github.com:iamzifei/wechat-article-formatter-skill.git
```

Or via HTTPS:

```bash
claude skill add https://github.com/iamzifei/wechat-article-formatter-skill.git
```

## Usage

Invoke the skill in Claude Code:

```
/wechat-article-formatter
```

Then provide either:
- A path to your markdown file: `/path/to/my-article.md`
- Or paste markdown content directly

### Example

```
/wechat-article-formatter

Please format my article at ~/articles/tech-post.md
```

The skill will:
1. Read your markdown file
2. Apply custom WeChat-optimized styling
3. Call the bm.md API to render the HTML
4. Save the formatted HTML to the same directory
5. Offer to publish as a draft to your WeChat account

## Output

The formatted HTML includes:
- Inline CSS styles (no external dependencies)
- WeChat-compatible markup
- Properly styled headings, paragraphs, lists, and code blocks
- Responsive design for mobile reading

## Customization

### Modifying Styles

Edit `styles/custom.css` to customize the appearance:

- **Primary color**: `rgb(53, 179, 120)` (green accent)
- **Font family**: Optima, Microsoft YaHei, PingFangSC-regular
- **Base font size**: 16px (body), 15px (paragraphs)

### API Configuration

The skill uses the following bm.md API settings:

| Parameter | Value | Description |
|-----------|-------|-------------|
| `codeTheme` | GreenSimple | Code block highlighting theme |
| `platform` | wechat | Output optimized for WeChat |
| `enableFootnoteLinks` | true | Convert links to footnotes |
| `openLinksInNewWindow` | true | Links open in new window |

## Integration

This skill can work with the [WeChat Article Publisher](https://github.com/iamzifei/wechat-article-publisher-skill) skill to directly publish formatted articles as drafts to your WeChat public account.

## Project Structure

```
wechat-article-formatter-skill/
├── skill.md          # Skill definition and instructions
├── styles/
│   └── custom.css    # Custom CSS for article styling
└── README.md         # This file
```

## Requirements

- Claude Code CLI
- Internet access for bm.md API calls

## License

MIT License

## Author

Created for use with Claude Code.

## Related

- [bm.md](https://bm.md) - Markdown rendering service
- [wechat-article-publisher-skill](https://github.com/iamzifei/wechat-article-publisher-skill) - Publish articles to WeChat
