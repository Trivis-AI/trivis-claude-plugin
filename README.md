# Trivis for Claude

Trivis is bookkeeping for Estonian companies. This plugin connects Claude to one
of your Trivis companies and teaches it how to work there: sales invoices drafted
from plain language and reviewed in a card inside the conversation, supplier
invoices read and checked before anyone posts them, and questions about
receivables, payables, cash flow and the ledger answered from the books.

## How to use it

1. Install the plugin from the Claude directory, or upload this folder as a ZIP
   under Customize > Plugins. In Claude Code:

   ```
   /plugin install trivis --marketplace Trivis-AI/trivis-claude-plugin
   ```
2. Open the plugin's Connectors tab and connect Trivis. You sign in with your
   Trivis account and choose the company. The connection is bound to that
   company and to the access level of your role there.
3. Ask. Every write is a draft until you confirm it. Anything that leaves the
   company, an email, an e-invoice, a payment run, waits for a person's approval
   in Trivis.

Works in English and Estonian.

## What data it sends

What you ask, and the text of documents you attach, goes to Trivis's API at
`ai-api.trivis.ee` under your own account and nowhere else. Claude cannot hand
files to Trivis, so the purchase batch card has a drop zone for the original
documents. See the [privacy policy](https://trivis.ee/en/privacy) and the
[terms of service](https://trivis.ee/en/terms).

## Contents

- `.claude-plugin/plugin.json`: the plugin manifest.
- `.mcp.json`: the Trivis connector, `https://ai-api.trivis.ee/mcp`.
- `skills/trivis-bookkeeping/SKILL.md`: how Claude works in Trivis.

The server behind the connector and the cards it renders are not in this
repository.

## Support and licence

Support: https://trivis.ee/contact. Licensed under the MIT licence, see
`LICENSE`. The Trivis name and logo are trademarks of Trivis OÜ and are not
covered by the licence.
