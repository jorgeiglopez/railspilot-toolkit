# Progress report email template

Canonical sample output. Match this structure and tone when drafting the client email.

```
Hi [Client]! Here's [Month]'s progress:

1. [Task Tag] <feature description> #<pr>
   https://github.com/<org>/<repo>/issues/<issue>

2. [Task Tag] <feature description> #<pr>
   https://github.com/<org>/<repo>/issues/<issue>

3. <feature description> (#<pr>, #<pr>)
   - <sub-item from related PR>
   - <sub-item from related PR>

N. Dev improvements:
   - <item> (#<pr> or <sha>)
   - <item> (#<pr>)

<one-line forward-looking note>

Thank you!

[Name].
```

## Format notes

- Numbered list, warm and specific. Professional but colleague-to-colleague.
- Prefix features with task tags when present in PR titles or linked issues (`[Epic]`, `[Athena]`, etc.).
- One issue URL per headline feature when resolvable from the PR body. Do not guess issue numbers.
- Consolidated smaller items share one numbered slot with sub-bullets.
- Dev improvements are the last numbered item, not mixed into feature slots.
- Optional closing sentence with an honest feature count (no invoicing block, no 12-feature baseline framing).
