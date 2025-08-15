---
title: "Wrangling with Transcript Files for LLMs"
date: 2025-08-14
categories: [GenAI, Development]
tags: [python, data wrangling]
image:
  path: /assets/img/posts/2025-08-14-Wrangling-with-Transcript-Files-for-LLMs.png
---
# Clean Your Microsoft Teams Transcripts for LLMs: Introducing VTTCleaner

## Introduction

In the age of remote work and virtual meetings, platforms like Microsoft Teams have become essential for collaboration. Teams offers automatic transcription of meetings, saving them in the VTT (WebVTT) format. While these transcripts are invaluable for record-keeping and analysis, their raw format is packed with metadata, timestamps, and verbose speaker tags. This makes them inefficient for direct use with Large Language Models (LLMs) such as GPT-4, Claude, or Gemini.

**VTTCleaner** is a Python tool designed to solve this problem. It transforms your Teams VTT transcripts into a clean, compact format, drastically reducing the token count and making them ready for LLM-powered analysis, summarization, and automation.

- [VTTCleaner GitHub Repository](https://github.com/warnov/vttcleaner)


---

## Why Do You Need VTTCleaner?

### The Problem

- **Raw VTT files are noisy:** They contain timestamps, speaker tags, and other metadata that are not useful for semantic analysis.
- **Token cost matters:** LLMs charge (or limit) based on the number of tokens. Extra metadata means higher cost and less room for actual content.
- **Speaker grouping:** Teams transcripts often split a single speaker’s message into multiple blocks, making it harder to analyze context or summarize conversations.

### The Solution

VTTCleaner:
- Removes timestamps and metadata.
- Simplifies speaker names to "FirstName + Initial".
- Groups consecutive lines from the same speaker.
- Handles lines without speaker tags, so no content is lost.
- Outputs a clean transcript, ready for LLMs.

---

## How VTTCleaner Works

### Input: Raw Teams VTT Transcript

Here’s a sample snippet from a typical Teams VTT file:

```
WEBVTT

96c14169-de06-4080-b9fe-f573f75eb719/91-0
00:00:04.409 --> 00:00:09.359
<v Martha Helena Calder&#243;n Franco>Sé que Juan no porque está en en Perú.
Perdón en Ecuador ya lo cambió de país y</v>

96c14169-de06-4080-b9fe-f573f75eb719/91-1
00:00:09.359 --> 00:00:12.948
<v Martha Helena Calder&#243;n Franco>de hecho está en una reunión con
Microsoft allá entonces,</v>

96c14169-de06-4080-b9fe-f573f75eb719/109-0
00:00:30.569 --> 00:00:31.089
<v Oscar Gutierrez>Bien.</v>
```

### Output: Cleaned Transcript

```
MarthaH: Sé que Juan no porque está en en Perú. Perdón en Ecuador ya lo cambió de país y de hecho está en una reunión con Microsoft allá entonces,
OscarG: Bien.
```

Notice how:
- Timestamps and IDs are gone.
- Speaker names are simplified.
- Consecutive lines from the same speaker are grouped.
- The transcript is much shorter and easier to process.

---

## Step-by-Step Guide

### 1. Clone the Repository

```sh
git clone https://github.com/warnov/vttcleaner.git
cd vttcleaner
```

### 2. Prepare Your Environment

No dependencies are required beyond Python 3.x.

### 3. Run VTTCleaner

```sh
python main.py
```

When prompted, enter the path to your VTT file (e.g., `sample.vtt`).

### 4. Get Your Cleaned Transcript

VTTCleaner will create a new file with the same name as your input, but with the suffix `_cleaned.txt`. For example, if your input is `meeting.vtt`, the output will be `meeting_cleaned.txt`.

---

## How Does VTTCleaner Work?

The core logic of VTTCleaner:

- Reads the VTT file and skips the header.
- Iterates through each line, identifying speaker blocks and plain text.
- Uses regular expressions to extract speaker names and text.
- Simplifies speaker names to "FirstName + Initial".
- Groups consecutive lines from the same speaker.
- Handles lines without speaker tags as "Unknown".
- Writes the cleaned transcript to a new file.

### Key Code Snippet

```python
# ...existing code...
if '<v ' in line:
    # Collect all lines until </v> is found
    full_block = [line]
    while i + 1 < len(lines) and '</v>' not in line:
        i += 1
        line = lines[i].strip()
        full_block.append(line)
    # Extract speaker and text
    block_text = ' '.join(full_block)
    match = speaker_pattern.search(block_text)
    if match:
        # ...simplify speaker name and group text...
# ...existing code...
```

---

## Real-World Benefits

- **Save money:** Lower token count means lower cost for LLM APIs.
- **Better context:** Grouped speaker interactions make it easier for LLMs to understand and summarize.
- **No content loss:** All transcript lines are preserved, even those without speaker tags.
- **Ready for automation:** Use cleaned transcripts for meeting summaries, action item extraction, sentiment analysis, and more.

---

## Who Should Use VTTCleaner?

- **Teams users:** Anyone who records and transcribes meetings in Microsoft Teams.
- **Data analysts:** Preparing transcripts for NLP or LLM analysis.
- **Developers:** Automating meeting summaries or insights.
- **Researchers:** Studying meeting dynamics or conversation flow.

---

## Contribute

VTTCleaner is open source! Contributions, feedback, and feature requests are welcome.

---

## License

MIT License

---

## Final Thoughts

As LLMs become more integrated into our workflows, preparing your data efficiently is more important than ever. VTTCleaner helps you unlock the full potential of your meeting transcripts—faster, cheaper, and smarter.