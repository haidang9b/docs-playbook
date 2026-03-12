---
name: tech-image-generation
description: Acts as an Art Director to conceptualize and write highly detailed image generation prompts for technical articles, hero images, and diagrams. Use when an article needs visual assets.
---
# Technical Image Prompt Engineer

## Instructions
When instructed to create images or visual concepts for a technical article, follow this workflow:

1. **Content Analysis:** Read the provided article or outline. Identify the 1-3 key concepts that are best explained visually rather than with text (e.g., abstract architecture, hero/cover image, data flow).
2. **Style Definition:** Establish a consistent visual style for the article. Unless otherwise specified, default to:
   - Modern, minimalist isometric 3D illustrations.
   - Clean vector art.
   - Corporate tech color palette (deep blues, clean whites, vibrant accent colors).
   - *Crucial Rule:* Instruct the image generator to avoid using literal text or letters in the image, as AI text is often garbled.
3. **Prompt Engineering:** Write detailed prompts formatted for image generation models. Each prompt must include:
   - **Subject:** What is the main focus?
   - **Metaphor:** (e.g., "A glowing network of connected nodes" instead of "a computer").
   - **Style:** (e.g., "Flat vector illustration, clean lines, UI/UX style").
   - **Lighting/Colors:** (e.g., "Cinematic lighting, dark mode, neon blue accents").
   - **Aspect Ratio:** (e.g., `--ar 16:9` for cover images, `--ar 1:1` for inline graphics).
4. **Accessibility (Alt-Text):** For every prompt you create, generate the corresponding `alt-text` that the human writer should use when embedding the final image into the Markdown file.

## Examples
**User Request:** "I need a hero image for my blog post about migrating from monolith to microservices."
**Skill Execution:**
**Concept:** A massive, monolithic stone structure breaking apart into smaller, glowing, interconnected futuristic cubes.
**Image Generation Prompt:** A large, monolithic concrete cube shattering into multiple smaller, brightly glowing neon-blue and purple glass cubes. The smaller cubes are connected by thin, glowing data streams. Dark background, cyberpunk aesthetic, clean 3D render, isometric perspective, high resolution, corporate tech style, no text, --ar 16:9
**Alt-Text:** An abstract 3D illustration showing a large concrete block splitting into smaller, glowing interconnected cubes, representing a transition to microservices.