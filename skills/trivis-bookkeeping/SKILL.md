---
name: trivis-bookkeeping
description: How to do bookkeeping for a Trivis company through the Trivis tools. Use whenever the user asks about invoices, suppliers, customers, payments, VAT, the ledger or their company's numbers, or attaches a business document.
---

# Working in Trivis

Trivis is the user's bookkeeping. One connection is bound to one company and to
the access level of the user's role there. Answer in the user's language; the
tools speak English and Estonian terms are fine in replies.

## Start of a session

Call `get_plugin_context` first. It names the company, the person you act for,
their access level and which workflows are available. Show the user which
company you are working in before changing anything. If they want a different
company, they reconnect Trivis and choose it; there is no tool for switching.

## Conventions every tool shares

- Money, quantities and percentages are decimal strings: "90", "12.5", "20".
  Dates are YYYY-MM-DD. Records are addressed by UUID.
- VAT is resolved by Trivis from the company's registration and the
  counterparty's country. Never choose a rate yourself and never state a rate
  you were not given back by a tool.
- A write creates a draft. Posting to the ledger is a separate step
  (`confirm_invoice`, `confirm_purchase_invoice`, `confirm_ledger_entry`) and
  you do it only when the user asks for it in this conversation.
- Some actions leave the company (email, e-invoice, payment run) or destroy
  something. Those return an approval request instead of running. Relay the
  link and the exact retry the tool text describes; approving happens in
  Trivis, by a person, and you re-call the tool with the approval id.
- A tool error is a result, not a reason to retry blindly. Read it, fix the
  input, or tell the user what Trivis said.
- Never fabricate a number. If a figure is not in a tool result, say so.

## Create a sales invoice

1. Find the customer with `list_contacts`. Several matches: ask which one.
   None: confirm the details with the user, then `create_contact`.
2. `create_invoice_draft` with the customer id and all the rows in one call:
   description, quantity, unit price without VAT, optional unit and discount.
   Products from the catalogue: `search_catalog_items`, then
   `get_catalog_item_prices` for the unit price, and pass `item_id`. Generate a
   fresh UUID as `idempotency_key`; reuse it only to retry that exact call.
   Dates and the payment term: pass what the user said, otherwise leave them
   out. Trivis then dates the invoice today, applies the company's default
   payment term and counts the due date from the invoice date. Never invent a
   term; "pay within 7 days" is `payment_term_days: 7`, "on receipt" is 0. The
   result opens the editor card: the person can edit there while you talk. A
   price you do not have: ask, or take it from the catalogue. Never open the
   customer's earlier invoices in the editor to look one up; a card shows the
   invoice being worked on, and an old draft on screen reads as the new one.
3. Later changes go through `save_invoice_edits` against the `revision` you
   last saw, as a batch of operations. A `conflict` means the person or someone
   else saved meanwhile: read the `current` document it carries, then decide
   with the user. `get_invoice_editor` reads the document at any time.
4. Show rows, subtotal, VAT and total as Trivis calculated them, never your own
   arithmetic. `get_invoice_preview` renders the real document into the card.
   A unit from the company's list, a currency, a document type (offer or
   waybill) or a tax rate the user names by hand take their ids from
   `get_invoice_options`, read once per session. Leave the tax rate to Trivis
   unless the user names one. `sort_order` on a row changes its place.
5. Post only on request with `confirm_invoice`. Send only on request with
   `send_invoice_email` or `send_invoice_einvoice` (`get_delivery_options`
   first). If a send fails or its outcome is unknown, say so and do not create
   another invoice or resend on your own. A company may not have switched on
   sending from outside Trivis: the tool then says so and gives the invoice's
   page in Trivis. Tell the user in those words, do not retry, and give them
   the link so they send it there themselves in one click; an admin can
   switch it on under Members, Kept for people.

There is no other way to build or change a sales invoice here: the generic row tools
are not offered to this plugin, so every draft goes through the editor tools above.

## Work with an existing sales invoice

- `copy_invoice` makes a new draft from any invoice and opens it in the card.
  `delete_invoice_draft` removes a draft for good; a posted invoice is
  cancelled with `cancel_invoice` instead, and a wrong posted invoice is
  corrected with `credit_invoice`, which makes a draft credit note: open it
  with `get_invoice_editor` so the card shows it. Both ask for approval.
- `create_invoice_share_link` makes the public page of a posted invoice: a link
  where the recipient views, downloads and pays it by card. Give the user the
  link; sending it is their call.
- The card has buttons for confirm, cancel, credit note, sending, copy and
  delete, and runs those tools itself. What it did reaches you through the
  card's context (status, revision); do not redo it, and do not call
  `get_invoice_editor` to check: a second card opened before a change shows
  the old state, and the open card follows the invoice by itself.

## Bring in purchase invoices

1. Files attached to the message go to `start_invoice_batch` in one call,
   all of them. It opens the purchase batch card and nothing posts from it
   until the person chooses Post there, whatever the company's auto-confirm
   setting. Do not open each file separately and do not use the upload link
   for attached files; `request_invoice_upload` is for files that cannot be
   attached here and for migrations.
   When this host gives you no file links for the attachments, read each
   document yourself and send its complete text, never a summary, as one
   entry of `documents` in the same `start_invoice_batch` call. Those drafts
   have no original attached: the card says so, posting stays off for them,
   and the person adds the originals through the card or an upload link
   you ask for. Never post a draft made from text yourself, and never try
   to fetch or post to an upload link on the person's behalf.
2. The card follows the batch while files are being read. Do not call
   `get_invoice_batch` in a loop; call it when the person asks what the state
   is, or after you changed a draft. Tell them what needs attention from the
   tool's text: supplier, total, and the finding, one line per file.
3. Fix a draft the card marks "needs review" through `check_purchase_invoice`
   and `repair_purchase_invoice`, or the row tools; rows without an expense
   account through `confirm_import_accounts` after asking which account.
   Duplicates and mismatched totals are findings to report, not to fix
   silently.
4. Posting is the person's: they press Post in the card, which runs
   `confirm_purchase_invoice` per selected draft. Call it yourself only when
   they say in words to post a named invoice. Companies with approval rounds
   keep them; a confirmation request in chat does not replace them.
5. "These three" means the card's selection: it is in your context as the
   batch id and the selected invoice ids.

## Ask about the business

`get_ar_summary`, `get_ap_summary`, `get_overdue_invoices`,
`get_cashflow_forecast`, `get_revenue_summary`, `get_trial_balance`,
`get_account_balance`, `list_ledger_entries`. The forecast is bucketed by due
date; call it that, not a bank balance. Drill into documents with
`list_invoices` and `list_purchase_invoices` when the user asks what is behind
a figure.

## Not in this plugin

Do not present `calculate_payroll` as a payroll run; it returns an estimate and
says so. Do not use `db_query`. Do not mark a VAT report as filed unless the
user states they filed it with the tax authority. Do not cancel, credit or
delete anything without the user naming the document and confirming.
