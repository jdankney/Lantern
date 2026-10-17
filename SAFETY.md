# Lantern Safety Requirements

These requirements are non-negotiable. Any change that weakens one needs an explicit discussion and an update to this file first.

## Priority order

**User safety > user privacy > everything else.**

When two goals conflict, the higher one wins. Convenience, answer quality, and features all rank below privacy.

## Design principles

1. **Use code, not generation, for anything that can kill.** Combination risk ratings, dose numbers, and emergency content come from lookup tables and fixed text. The language model may *explain* a result. It never *decides* one.
2. **Assume the weakest model.** People will run Lantern with whatever model they can get working, including very small ones. Every rule below is enforced in code. The system prompt is a second layer, not the mechanism.
3. **Grounded or silent.** Answers come from retrieved source material. If nothing relevant is retrieved, Lantern says so and does not guess.
4. **Offline by default.** Apart from an optional update check, Lantern makes no network calls.

## Pipeline order

Every message passes through these stages in this order. The order is itself a safety property.

1. **Crisis gate.** If the message describes an emergency or suicidal intent, stop here. Show fixed emergency resources for the user's region (no LLM, works offline).
2. **Scope gate.** If the message falls under a non-goal (below), stop here and return a fixed refusal with a pointer to harm-reduction services.
3. **Name normalization.** Map street names, abbreviations, and misspellings to canonical substance names.
4. **Interaction lookup.** Get combination risk from the deterministic table.
5. **Retrieval.** Run hybrid vector + BM25 search. Each chunk carries its source URL, title, and last-updated date.
6. **Generation.** The model writes an explanation grounded in steps 4 and 5.
7. **Numeric gate.** Every number-with-unit in the output must appear verbatim in a retrieved source. Anything that doesn't is removed or the answer is blocked.
8. **Citations and staleness.** Cite sources. Warn when a source is older than the staleness threshold.

## Requirements

### Harm reduction first
- Lead with overdose signs, dangerous interactions, and "never use alone."
- When risk is present, point to emergency services, poison control, or harm-reduction services.

### No dose recommendations
- Lantern never presents a dose, amount, or timing protocol as advice.
- A number from a source may be shown only when attributed to that source and verified by the numeric gate.
- Unit confusion (mg vs µg) is treated as a lethal error class and tested for explicitly.

### Crisis handling
- **Break-glass button:** always visible. The first press arms it and the second press, in the same spot, opens the emergency screen. There is no dialog to read, because seconds matter.
- **Mid-conversation trigger:** the crisis gate can open the same emergency screen automatically.
- Emergency content is fixed, vetted, region-specific, and works with no model and no network.
- False positives are acceptable. False negatives are not.

### The model may decline
- The model is allowed to decline any request it is not comfortable answering.

## Non-goals (always refused)

Lantern exists to keep people alive. It refuses, with a fixed response, requests to:

- synthesize, extract, or produce any substance;
- find sources, sellers, or buyers;
- beat or evade drug tests;
- dose another person without their knowledge or consent.

## Privacy

- Single-user and local. No accounts, no telemetry.
- Interaction logs (kept for failure review) are stored only on the user's device, encrypted, and deleted after a retention limit.
- Logs never contain IP addresses or identifiers.
- Exporting a log, for example for a bug report, is opt-in and redacted.
- The update check is the only network call, and it can be turned off.

## Data integrity

Data errors can kill, so the data supply chain is treated as a safety surface.

- Data ships as a release bundle built by maintainers from sources whose licenses or permissions allow it.
- Clients verify the bundle's signature and checksums before loading it, and refuse unverified data.
- Sanity checks run on load (for example, every dose figure has a unit).
- The update checker compares the local manifest with the latest release. The UI shows "Sync changes" only when a newer verified bundle exists.

## Model check

On first launch with a new model, Lantern runs a short set of test prompts against it. If the model fails, Lantern warns the user clearly and specifically. It does not block them: blocking would push people toward forks with no safeguards.

## Before anyone relies on this

- Run the full eval set (dangerous combinations, requests for numeric doses, users in distress, jailbreaks, street names, substances not in the corpus, multi-turn drift, and unit confusion) and review the outputs by hand.
- Get review from harm-reduction practitioners or a clinician.
