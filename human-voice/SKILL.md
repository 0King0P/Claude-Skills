---
name: human-voice
description: >
  Any text that will be read as the user's own words — emails, messages, bios,
  posts, proposals, cover letters, announcements, reports. Strips out the
  tells that make writing sound AI-generated and matches the user's actual
  voice instead. Use whenever you're producing prose on behalf of the user.
  Composes with ultra-efficient.
---

# Human Voice

This skill is the one rule for any text that will be read as if the user wrote it. If a reader is going to assume these are the user's own words, the writing must not leak its AI origin. That's the whole job.

## The non-negotiable

All text output that represents the user's voice — emails, messages, documents, social posts, bios, reports, proposals, cover letters, anything that a recipient will read as if the user wrote it — must pass as genuinely human-written. This is not a stylistic preference. It's the requirement.

## Before you write

- **Who's the audience?** A colleague? A stranger? A VP? Friends? The tone is different for each.
- **Who's the user?** If you have samples of their writing, read them. Their cadence, their vocabulary, their favorite words, their punctuation habits — all of that is the target.
- **What's the goal of the message?** One-line what-am-I-doing-here. If you can't state it, you'll pad.
- **What's the single thing the reader needs to do or know after reading this?** Lead with that.

## Banned patterns — the AI tells

These scream "ChatGPT wrote this." Remove them on sight:

### Banned words (almost always wrong in user-voice writing)
- "Leverage", "utilize", "delve", "tapestry", "landscape"
- "Robust", "seamless", "cutting-edge", "game-changer", "synergy"
- "Holistic", "paradigm", "innovative", "revolutionize", "empower"
- "Foster", "nuanced", "multifaceted", "comprehensive", "streamline"
- "Elevate", "unlock", "transform", "bespoke", "curate"
- "Pivotal", "vibrant", "dynamic", "meticulous"

### Banned openers
- "In today's [fast-paced / ever-evolving / digital] world..."
- "In the ever-evolving landscape of..."
- "It's important to note that..."
- "This is a testament to..."
- "At the end of the day..."
- "As a [role/adjective], I..."
- "I'm excited to...", "I'm thrilled to...", "I'm passionate about..."
- "I hope this message finds you well" (outside strictly formal contexts)

### Banned connectives
- Sentences starting with "Furthermore", "Moreover", "Additionally", "Consequently", "Nevertheless" — real people rarely chain these
- "Not only X but also Y" constructions
- Overly smooth transitions between every paragraph

### Banned structural tics
- Em dash abuse — one per paragraph max, zero is fine
- Exclamation marks in professional writing (unless the user's style clearly uses them)
- Perfect parallel structure in every list (humans aren't that symmetrical)
- Triple adjective stacks ("innovative, dynamic, and forward-thinking")
- The setup-expand-conclude rhythm in every paragraph
- The "this not only X but also Y" rhythm
- The neat three-bullet list where two or five would be more natural

## What human writing actually sounds like

- **Uneven sentence length.** Short punchy ones mixed with longer ones. Not uniform.
- **Some sentences start with "And," "But," "So."** That's normal conversation.
- **Contractions.** Don't, won't, it's, can't, I'm. "Cannot" only when formality demands it.
- **Concrete numbers and facts.** "Cut load time from 3s to 400ms" beats "significantly improved performance."
- **Occasional bluntness.** "This didn't work" beats "this presented challenges."
- **Skips the thesis statement.** Gets to the point without announcing it.
- **Specifics over generalities.** Names of things, actual numbers, exact phrases.
- **Slight roughness.** Not every sentence flows perfectly into the next. Real writing has tiny bumps.
- **Voice.** Sounds like a person talking, not a press release or a LinkedIn post.

## Match the user

If you have any of the user's own writing in the conversation or in provided files, study it before you write:

- Sentence length distribution
- Vocabulary — formal or casual? Technical or plain?
- Punctuation habits — do they use em dashes? semicolons? exclamation marks?
- How do they start sentences?
- Do they swear? Make jokes? Use specific slang?
- Do they use bullet points or prose?
- Signatures, sign-offs, greetings — what do they actually write?

Mirror them. Don't "improve" their voice — replicate it. "Improved" voice reads wrong to everyone who knows the user.

If you don't have samples, ask for one or lean into a neutral human tone: direct, specific, contraction-using, short-sentence-friendly.

## The pre-send check

Before outputting any user-voice text, run this:

1. **Read it aloud.** Does it sound like a real person talking, or like a pitch deck? Rewrite anything that sounds like a pitch.
2. **Pattern scan.** Any of the banned words or openers? Strip them.
3. **Rhythm scan.** Is every sentence the same length? Break it up.
4. **Parallelism scan.** Did every bullet magically end up the same length and structure? That's AI. Make one shorter or longer.
5. **Specificity scan.** Are there vague claims that could be made concrete? Replace "significantly" with an actual number.
6. **Confidence scan.** Any unnecessary hedging ("I think maybe we could possibly...")? Cut.
7. **Opener scan.** Does it start with the point, or with preamble? Kill the preamble.

## Length

Respect the reader's time. A three-line email beats a three-paragraph one that says the same thing. The moment you're padding, stop.

Guidelines:
- Quick ask → 1-3 sentences
- Status update → one short paragraph
- Proposal / pitch → only as long as it needs to be, with the ask in the first line
- Cover letter → under 250 words unless the user specifies otherwise
- Social post → the shortest version that still makes the point

## Different contexts

### Emails
- Subject line that's the point, not a teaser
- First line has the ask or the news
- Context only if needed
- Sign-off the user actually uses

### Slack / chat
- Much more informal than email
- Shorter sentences
- Contractions, lowercase starts if that's the user's style
- No "Dear ...", no "Best regards"

### Social (LinkedIn, Twitter, posts)
- Warning: LinkedIn is the epicenter of AI-sounding writing. Do NOT drift toward LinkedIn voice unless the user specifically wants it.
- Lead with the hook, not the setup
- Avoid "I'm thrilled to announce"
- Short lines, real words, no humblebrag

### Cover letters / applications
- Specific to this role, not generic
- Concrete accomplishments with numbers
- First paragraph: why this job, not "I am writing to apply for..."
- Skip "As a passionate X with Y years of experience..."

### Internal docs / status reports
- Facts first, interpretation second
- Numbers and dates, not vague direction
- Bullet points are OK here, but not everywhere
- No "stakeholders" unless you mean it literally

## Edge case: the user asks for "professional" tone

"Professional" to most people does not mean "AI-stilted corporate prose." It means: clear, respectful, appropriate for the audience, free of slang they might not share. It does NOT mean: "leverage," "robust," "holistic," or "I hope this finds you well."

If the user asks for formal, keep it formal — but still human. Short sentences, clear asks, zero fluff. Formal and AI-sounding are not the same thing.

## Activation

When this skill activates, respond with:

🗣️

Then ask for samples of the user's voice (if you don't have any) and start writing.
