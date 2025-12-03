# Theme Overrides

This folder contains custom layouts and shortcodes that override or extend the [Hugo Stack theme](https://github.com/CaiJimmy/hugo-theme-stack).

## Custom Shortcodes

### `img` - Resizable Image with Gallery Support

A shortcode for displaying images with height-based sizing control while maintaining PhotoSwipe gallery integration.

#### Usage

```markdown
{{< img src="image.webp" alt="Description" height="50vh" >}}
```

#### Parameters

| Parameter | Required | Description |
|-----------|----------|-------------|
| `src` | Yes | Image filename (must be in the same page bundle as the post) |
| `alt` | No | Alt text for the image. Also displayed as the caption below the image |
| `height` | No | Maximum height using any CSS unit (e.g., `50vh`, `300px`, `20rem`). The image scales proportionally |
| `class` | No | Additional CSS classes to add to the figure element |

#### Features

- **Relative Height Control**: Use viewport-relative units (`vh`) for consistent sizing across screen sizes
- **PhotoSwipe Gallery**: Clicking the image opens it in the Stack theme's PhotoSwipe lightbox at full resolution
- **Lazy Loading**: Images use `loading="lazy"` for better page performance
- **Caption from Alt**: The `alt` text is displayed as a `<figcaption>` below the image
- **Aspect Ratio Preserved**: Width scales automatically to maintain the original aspect ratio

#### Examples

**Mobile app screenshot (50% viewport height):**

```markdown
{{< img src="screenshot.webp" alt="App screenshot" height="50vh" >}}
```

**Fixed pixel height:**

```markdown
{{< img src="diagram.webp" alt="Architecture diagram" height="300px" >}}
```

**Full size (no height constraint):**

```markdown
{{< img src="photo.webp" alt="Full size photo" >}}
```

**With custom class:**

```markdown
{{< img src="logo.webp" height="10rem" class="my-custom-class" >}}
```

#### Comparison with Standard Markdown

| Feature | `![alt](image.webp)` | `{{< img src="image.webp" >}}` |
|---------|---------------------|-------------------------------|
| Height control | No | Yes, via `height` parameter (any CSS unit) |
| Gallery support | Yes | Yes |
| Responsive sizing | Fixed srcset widths | Viewport-relative with `vh` |
| Caption | From alt text | From alt text |

Use standard markdown syntax when you don't need size control. Use the `img` shortcode when you want to limit the display height of images (e.g., for tall mobile app screenshots that would otherwise dominate the page).

## Existing Overrides

### Layouts

- `layouts/_default/_markup/render-codeblock.html` - Mermaid diagram support
- `layouts/partials/head/custom.html` - Analytics, BMC widget, custom CSS variables, Mermaid JS
- `layouts/partials/head/head.html` - Meta tags, styles, scripts
- `layouts/partials/footer/components/custom-font.html` - Custom font loading
