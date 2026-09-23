# Filing decision records

Run only after the user has named the records to file. For each one:

1. If `docs/adr/` does not exist, run `adrs init docs/adr` once. It
   creates record 1, "Record architecture decisions". Append to its
   Decision section: records describe what was decided and on what facts;
   they are never a reason to refuse a change; a conflicting change is
   raised with the person deciding and, if chosen, written as a
   superseding record.
2. Run:

   ```
   adrs new --no-edit --format nygard --variant bare --status accepted "<title>"
   ```

   Add `--supersedes <N>` when it replaces record N; that also marks N as
   superseded. The command prints the file path and opens no editor.
3. Edit that file. Fill the three sections in prose, in the language the
   interview was held in, no headings inside them, no fixed layout:
   - Context: the question that raised the decision; the facts it rested
     on, how they were verified, and what was not; the directions
     considered and what each gave up; that the user decided it in this
     interview, on this date.
   - Decision: what the user chose, in their words.
   - Consequences: what it gives up, what changes for existing behavior,
     how to reverse it, and under what change in facts to reconsider it.

   Nothing about implementation effort; nothing that was not in a brief or
   an answer.
4. Report each file created with number and title, and end. Run nothing
   else; touch no other file.
