# SVG Configuration Example (Default)

Example configuration for using SVG generation, the default and recommended provider.

**Note**: SVG generation ALWAYS produces both `.svg` (editable source) and `.jpg` (for GitHub upload).

## Configuration

```yaml
# .claude/github-social.local.md
---
provider: svg
svg_style: minimal
dark_mode: false
output_path: .github/social-preview.svg
dimensions: 1280x640
include_text: true
colors: auto
upload_to_repo: false
---
```

## No API Key Required

SVG generation is performed directly by Claude - no external API calls needed.

## SVG Style Options

| Style | Description | Shapes | Best For |
|-------|-------------|--------|----------|
| `minimal` | Clean, focused | 3-5 | Professional, understated |
| `geometric` | More complex | 8-15 | Technical projects |
| `illustrated` | Organic, warm | Varies | Approachable projects |

### Minimal (Default)
- Project name as focal point
- 3-5 subtle accent shapes
- 60%+ whitespace
- Single gradient or solid background

### Geometric
- Grid or flow-based arrangements
- Abstract domain metaphors
- More visual interest

### Illustrated
- Organic SVG paths with curves
- Hand-drawn aesthetic
- Warmer color palettes

## Dark Mode Support

```yaml
# Generate both light and dark variants
dark_mode: both
```

This creates:
- `.github/social-preview.svg` (light SVG)
- `.github/social-preview.jpg` (light JPG - for GitHub upload)
- `.github/social-preview-dark.svg` (dark SVG)
- `.github/social-preview-dark.jpg` (dark JPG)

## Expected Behavior

When running `/social-preview`:

1. Project files are analyzed
2. Domain is detected (DevTools, AI, Web, etc.)
3. Appropriate color palette is selected
4. SVG is generated using domain template
5. SVG is saved to `.github/social-preview.svg`
6. **JPG is generated** from SVG (1280x640, quality 90)
7. JPG is saved to `.github/social-preview.jpg`
8. Both file locations and sizes are reported

**Output Files**:
- `.github/social-preview.svg` - Editable source (10-50KB)
- `.github/social-preview.jpg` - For GitHub upload (<1MB)

## Benefits of SVG

| Feature | Value |
|---------|-------|
| Cost | Free |
| Speed | Instant |
| SVG File Size | 10-50KB |
| JPG File Size | 50-200KB |
| Editability | Full (SVG is text-based) |
| Consistency | 100% predictable |
| Dark Mode | Easy variants |
| GitHub Compatible | JPG always generated |

## Domain Color Palettes

The SVG generator automatically selects colors based on project domain:

**DevTools/CLI**:
- Background: #0f172a → #1e293b
- Accent: #06b6d4 (cyan)

**AI/ML**:
- Background: #1e1b4b → #312e81
- Accent: #8b5cf6 (violet)

**Web/Frontend**:
- Background: #0c4a6e → #075985
- Accent: #3b82f6 (blue)

**Data/Analytics**:
- Background: #1e3a5f → #0f172a
- Accent: #f59e0b (amber)

**Security**:
- Background: #0f172a → #1e293b
- Accent: #22c55e (green)

**Infrastructure**:
- Background: #1e293b → #334155
- Accent: #06b6d4 (cyan)

## Customizing Output

You can edit the generated SVG directly:
- Open in any text editor
- Modify colors, text, shapes
- No regeneration needed

## Example Output

A minimal SVG for a DevTools project might look like:

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1280 640">
  <defs>
    <linearGradient id="bg" x1="0%" y1="0%" x2="100%" y2="100%">
      <stop offset="0%" style="stop-color:#0f172a"/>
      <stop offset="100%" style="stop-color:#1e293b"/>
    </linearGradient>
  </defs>
  <rect width="1280" height="640" fill="url(#bg)"/>
  <circle cx="640" cy="260" r="100" fill="#06b6d4" opacity="0.15"/>
  <text x="640" y="340" font-family="-apple-system, sans-serif"
        font-size="72" font-weight="700" text-anchor="middle" fill="#f8fafc">
    project-name
  </text>
  <text x="640" y="400" font-family="-apple-system, sans-serif"
        font-size="24" text-anchor="middle" fill="#94a3b8">
    Your project tagline
  </text>
</svg>
```

## When to Use AI Instead

Consider switching to DALL-E or Gemini when:
- You want artistic, creative imagery
- You need photorealistic elements
- SVG aesthetic doesn't fit your brand
- You want variety between regenerations

Override with: `--provider=dalle-3` or `--provider=gemini`
