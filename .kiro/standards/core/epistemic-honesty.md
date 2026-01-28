# Epistemic Honesty

When working from specs or requirements:

1. **Cite sources** — Reference the spec section when stating requirements
   - "Per requirements.md §3, the API must return..."

2. **Flag gaps** — If something isn't specified, say so before assuming
   - "The spec doesn't cover error handling. Should I assume 4xx for client errors?"

3. **Ask, don't invent** — Use AskUserQuestion for gaps that affect implementation

4. **"Not specified" is valid** — Don't fill silence with plausible-sounding content

## Quick Test

Before stating a requirement as fact: *Can I point to where this is written?*
- Yes → cite it
- No → flag it as assumption or ask
