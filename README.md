# ProofDeck

A private study guide that turns your notes into flashcards and quiz questions. Every AI-written card waits **beside the sentence it came from** until you approve it.

**Live:** https://nickmaingott.github.io/proofdeck/

## How it works

1. **Add notes.** Paste text or upload a `.txt` / `.md` file. The built-in sample is my own notes on real-time multiplayer networking, written while building [Shard Scramble](https://nickmaingott.github.io/shard-scramble/).
2. **Generate.** Claude (Opus 5.5 by default) reads the notes and returns flashcards and multiple-choice questions as JSON. Each card includes an `excerpt`: a sentence quoted word-for-word from the notes. Without an API key, an offline extractor pulls "**Term** — definition" lines instead.
3. **Review (human in the loop).** Each draft is shown next to the matching sentence in your notes, highlighted, with an automatic match check:
   - ✓ **Exact quote**: the excerpt appears in your notes.
   - ≈ **Close match**: similar wording, so compare carefully.
   - ✗ **Not found**: the AI's quote isn't in your notes, so the card probably isn't grounded.

   You approve, edit, or discard every card. Nothing reaches the library automatically.
4. **Library & quiz.** Approved cards are searchable and power flashcard and multiple-choice quizzes. Missed cards come back first.

## Privacy

- **Passcode lock:** all data (notes, cards, API key) is encrypted in the browser with AES-GCM, using a key derived from your passcode with PBKDF2 (310,000 iterations). It's stored in `localStorage` on your device. There is no server.
- **Auto-lock** after 15 minutes of inactivity.
- **Bring your own key:** the Anthropic API key is stored inside the encrypted vault and sent only to `api.anthropic.com`.
- Encrypted backup download and restore.

## Tech

A single static `index.html` (vanilla JS and the Web Crypto API) hosted on GitHub Pages. It uses the official Anthropic JS SDK with structured JSON output (`output_config.format`) and server-side refusal fallbacks.

## Run locally

```bash
python3 -m http.server 8766
```
