# {{COMPANY}} brand pack

Version {{BRAND_PACK_VERSION | 1.0}} · Updated {{BRAND_PACK_DATE}} · Owner {{OWNER_NAME}}

> Build agent: fill this with the person in Phase 4 of the kickoff. Every value comes from them, their files, or their existing brand guide. Ask the question in each _Ask_ line; delete the _Ask_ lines and this note when the pack is filled. Expand the `{{#if}}` and `{{#each}}` blocks and renumber the hard copy rules. Keep the finished pack under 40,000 characters and free of emoji. The first line after the title must stay `Version X.Y`, and the last line must stay exactly `END OF BRAND PACK`: routines check both on every run. Raise the version on every change and record it in the brief's change log.

What every design, social post, video, and article image follows. Built from: {{BRAND_SOURCES}}. Where this pack and its sources disagree, {{SOURCE_OF_TRUTH | the sources win, and this pack is corrected}}.

Read by: `CP · Copy Writer` (voice and hard copy rules, every run), `CP · Creative Drafts` (hard copy rules and imagery, every run), the people who finish designs, video and voiceover production, and CMS drafts.

## Colour

_Ask: the brand colours as hex values, and the role of each. Read them from the design system or CSS tokens if Phase 0 found one._

| Role                   | Hex                          | Use                                                                   |
| ---------------------- | ---------------------------- | --------------------------------------------------------------------- |
| **Primary**            | `{{COLOUR_PRIMARY}}`         | Brand colour. Headlines on light backgrounds, primary buttons, badges |
| Primary scale          | `{{COLOUR_PRIMARY_SCALE}}`   | Darkest to lightest: pressed, hover, highlight, tint                  |
| **Secondary**          | `{{COLOUR_SECONDARY}}`       | Secondary calls to action, information                                |
| Secondary scale        | `{{COLOUR_SECONDARY_SCALE}}` | Darkest to lightest                                                   |
| **Accent**             | `{{COLOUR_ACCENT}}`          | {{ACCENT_USE \| Marketing emphasis only. Never body copy}}            |
| **Social backgrounds** | `{{COLOUR_SOCIAL_BG}}`       | Full-bleed backgrounds for social templates                           |
| Error                  | `{{COLOUR_ERROR}}`           | Errors and warnings only. Never decorative                            |
| Neutral background     | `{{COLOUR_NEUTRAL_BG}}`      | Light section backgrounds, illustration base                          |
| Text                   | `{{COLOUR_TEXT}}`            | Primary / secondary / tertiary                                        |

Contrast rule: {{CONTRAST_RULES}}. _Ask: which text colour goes on which background? Check each pair against WCAG AA (4.5:1 for body text, 3:1 for large text) and write only passing pairs here. List the pairs that must never meet._

## Type

_Ask: the display and body fonts, and whether the design tool has them built in._

| Use                      | Font               | Weight                     | Tracking                           | Line height                    |
| ------------------------ | ------------------ | -------------------------- | ---------------------------------- | ------------------------------ |
| Display / H1             | `{{FONT_DISPLAY}}` | `{{WEIGHT_H1 \| 800}}`     | `{{TRACKING_HEADINGS \| -0.02em}}` | `{{LINE_HEIGHT_H1 \| 1.1}}`    |
| H2                       | `{{FONT_DISPLAY}}` | `{{WEIGHT_H2 \| 700}}`     | `{{TRACKING_HEADINGS \| -0.02em}}` | 1.2                            |
| H3 / H4                  | `{{FONT_DISPLAY}}` | `{{WEIGHT_H3 \| 600–700}}` | -0.015em                           | 1.3                            |
| Labels and badges (caps) | `{{FONT_DISPLAY}}` | 700                        | +0.1em                             | —                              |
| Body                     | `{{FONT_BODY}}`    | `{{WEIGHT_BODY \| 400}}`   | 0                                  | `{{LINE_HEIGHT_BODY \| 1.55}}` |
| Captions                 | `{{FONT_BODY}}`    | `{{WEIGHT_BODY \| 400}}`   | +0.02em                            | 1.45                           |

Font files, for tools that need them: {{FONT_FILES}}. _Ask: where are the font files, and is the licence good for social and video use?_

## Logos and patterns

_Ask: every logo variant, where each lives in the design tool, its public URL, and the backgrounds it may sit on._

| Asset           | Design tool id     | Public URL     | Use on            |
| --------------- | ------------------ | -------------- | ----------------- |
| `{{LOGO_NAME}}` | `{{LOGO_TOOL_ID}}` | `{{LOGO_URL}}` | `{{LOGO_USE_ON}}` |

Repeat the row once per logo in the brief (`LOGOS`): one per variant (horizontal lockup on light, lockup on dark, mark only, square avatar, the scalable master) and one per background pattern. A variant with no public URL is uploaded to the design tool by hand before any template uses it.

**Logo rules:**

- Clear space on every side: {{LOGO_CLEAR_SPACE | the height of the mark's capital letter}}.
- Minimum width: {{LOGO_MIN_WIDTH | 120 px for the lockup, 32 px for the mark}}.
- Never recolour outside the variants above, stretch, add effects, or place the logo on a busy photo.
- {{NAME_IN_CAPS_RULE | The name in capitals appears only inside the logo. Running text writes the name as in Block 0.}}

## Imagery

_Ask: photography, illustration, or both? Are AI-generated images allowed, and of what (never of real or realistic people, unless they say otherwise)? Where are the approved illustrations?_

- Style: {{IMAGERY_STYLE}}.
- Generated imagery: {{GENERATED_IMAGERY_RULE | no realistic people, no photorealism; any synthetic media is marked on its Asset}}.
- Palette: led by the colours above.
- Approved library: {{IMAGE_LIBRARY}}.
- Leave a calm field for text: on covers, art sits to one side and the headline gets the other ~55%.
- Product screens in a video or carousel sit in a phone frame (9:16) or a browser window (16:9), using real screens.

## Voice and tone

_Ask: three to five voice rules, each with one sentence that shows it. A sentence they love, and one they would never post._

{{VOICE_RULES}}

- **Calls to action are actions:** {{CTA_EXAMPLES}}.
- **Spelling:** `{{LANGUAGE}}`. {{SPELLING_EXAMPLES}}
- **Geography:** {{GEOGRAPHY}}.
- **Emoji:** {{EMOJI_RULE | none in product UI; at most one per social post, never in place of words}}.
- **Hashtags:** {{HOUSE_HASHTAGS}}, plus 1–3 topical tags where the channel's Hashtag Style allows. **None on:** {{NO_HASHTAG_CHANNELS}}.
- **Personal accounts** ({{PERSONAL_CHANNELS}}) speak in their owner's first person, never as the brand page.

### Hard copy rules (every template, prompt and post)

These rules are binding. Routines also carry them inline. If this section and a routine disagree, the routine wins until this pack is corrected.

The rules apply to every caption, every word on a design, every video caption, and every alt text. The ones that depend on the brief come last, so removing one leaves no gap to renumber.

1. **No live numbers.** Never type {{NEVER_TYPE_NUMBERS}}: no totals, counts, or percentages from our own data. They drift the day after posting. Describe the landscape in terms that stay true, and cite external figures with their source.
2. **No sensitive meeting content:** no names of people outside {{COMPANY}}, no money, funding, or grant terms, nothing personal, no partners that have not been announced, no quotes from meetings.
3. **Every post and every design goes through human approval.** Approvers: {{APPROVERS}}. A personal account is approved only by its owner. A design or file is approved on its own, in Asset Review.
4. **AI labels only where a platform requires them** (realistic synthetic video, voice, or images). No caption disclaimers.
5. **Never say:** {{NEVER_SAY}}.
6. **Claims:** {{CLAIMS_RULES | superlatives ("best", "#1", "leading") and guarantees only with a cited, current source}}.
   {{#if APPROVED_WORDINGS}}7. **Approved wordings, verbatim.** {{APPROVED_WORDINGS}}. For each: the exact short form, the exact long form, the fine print, what triggers it, and who approved it on which date. Never paraphrase, never change a number, never mix two forms in one post.{{/if}}
   {{#if HELD_SERIES}}8. **Held series.** Nothing in {{HELD_SERIES}} publishes until {{HOLD_UNTIL}}. It may be drafted and approved; the gate holds it.{{/if}}
   {{#if REGULATED_TOPICS}}9. **Regulated topics:** {{REGULATED_TOPICS}}. Each uses this disclaimer: {{DISCLAIMER_TEXT}}, and is reviewed by {{REGULATED_REVIEWER}} before approval.{{/if}}

### Do and don't

_Ask for at least five pairs, with one pair for each hard copy rule. Use their real product language; never a real customer's name._

| Do             | Don't                      |
| -------------- | -------------------------- |
| {{DO_EXAMPLE}} | {{DONT_EXAMPLE}} ({{WHY}}) |

## Channel sizes and safe zones

Platform facts, current as of 2026. Check a platform's help page when a design looks cropped.

| Channel                                           | Canvas                                      | Keep text and UI inside                                                    |
| ------------------------------------------------- | ------------------------------------------- | -------------------------------------------------------------------------- |
| Instagram / Facebook feed                         | 1080×1350 (4:5) or 1080×1080                | 60 px margin                                                               |
| Instagram Reels / Stories, TikTok, YouTube Shorts | 1080×1920 (9:16)                            | Out of the top 220 px, the bottom 380 px, and 90 px at each side           |
| YouTube long-form, LinkedIn video, website hero   | 1920×1080 (16:9)                            | 5% title-safe (96×54 px). Recompose for 16:9; don't letterbox the 9:16 cut |
| YouTube thumbnail                                 | 1280×720                                    | Nothing important in the bottom-right 200×80 px (the timestamp)            |
| LinkedIn image / document carousel                | 1080×1350 or 1200×1200; PDF pages 1080×1350 | 60 px margin                                                               |
| X image                                           | 1600×900 or 1080×1080                       | Centre-weighted; previews crop to 16:9                                     |
| Google Business update                            | 1200×900 (4:3), minimum 400×300             | Centre-weighted. Text only or a single photo, no video                     |

## Designs from templates

What the creative step may and may not do. A person finishes and approves every design.

- **Templates first.** Drafts are new designs made from the brand templates in the Templates table, one per channel and kind. A template is changed only by a person, and only in the design tool.
- **Words on a design come from the post.** The title, the hook, the caption's first line, a published quote, the canonical URL, or a fixed text the template's Field Map names. Calls to action on designs: {{CTA_EXAMPLES}}.
- **Images come from the brand kit or an approved asset.** {{GENERATED_IMAGERY_RULE | No realistic generated people, no photorealism}}. A design with realistic generated imagery is marked Synthetic Media on its asset.
- **Leave room.** Keep text inside the channel's safe zone (the table above). One message per design; a carousel tells one idea per page.
- **What a person finishes:** image choice and crop, anything the template could not lay out, and the final check against this pack. Then they approve the asset.

## Video and audio

- 30 fps. Captions on by default: design every video to work with the sound off.
- Loudness: {{LOUDNESS_TARGET | -14 LUFS integrated, true peak at or below -1 dBTP}} for social.
- Voice: {{VOICE_POLICY | human voices; a cloned voice only with its owner's written consent, and marked as synthetic on its Asset}}.
- End card: {{END_CARD}}.
- **A person finishes every cut and approves every read.** Editors' agents may import, rough-cut, caption, and remove filler; a voice tool may draft a read. None of it is final until a person says so on the asset.

## Design tool brand kit: setup

A person does this once in `{{DESIGN_TOOL | Canva}}`. Most design tools' APIs cannot create a brand kit, so it is a manual step. About ten minutes for the kit, then a few minutes per template. Brand kits and brand templates need a plan that has them (Canva: a team plan): check before you start.

1. Open the brand area (Canva: **Brand** → **Create a Brand Kit**) and name the kit `{{COMPANY}}`.
2. **Logos:** add every variant from the logo table.
3. **Colours:** make three palettes.
   - "Core": the primary colour and its scale.
   - "Secondary": the secondary colour and its scale.
   - "Social": the social backgrounds, the accent, the neutral background, the text colour, and white.
4. **Fonts:** Title = display font, heaviest weight; Subtitle = display font, bold; Heading = display font, semibold; Body = body font, regular; Quote = display font, medium italic if available, else body font, medium.
5. **Brand graphics:** add the patterns and the approved illustrations.
6. **Brand voice** (if the tool offers it): paste the Voice and tone section above.
7. **Brand templates.** For each channel and kind in the brief's template map, make a design at the channel's size from the kit, put a placeholder text in each spot the copy goes (for example `HEADLINE TEXT`, `QUOTE TEXT`), and publish it as a brand template. Where the plan supports data fields, tag those spots as fields instead. The placeholder texts and field names go into the template's Field Map.
8. Tell the agent when it is done. It reads each template's id and fields, fills the Templates table, and you check the Field Maps together.

END OF BRAND PACK
