# Experiment 01: recognize generated SQLite columns

Hypothesis: the database inspector falsely rejects valid stored generated columns
because `PRAGMA table_info` omits them.

Only production change: use `PRAGMA table_xinfo` for the column-name inventory.
The schema, generated SQL, prompts, model and retry policy are unchanged.

Replay source: BookStack run `bookstack-local-resumed-20261107-205034`, application
checkpoint `2141c12` (the first database repair, before the second repair replaced
the generated column with an ordinary required column). Detailed local evidence
is in `runs/experiment-generated-columns-20261107/` in the enclosing workspace.

| Check | Before | After |
| --- | --- | --- |
| Identical historical SQL and schema | Missing column: RecentlyViewed.unique | All seven database initialization/startup checks pass |
| Independent stored generated column regression | Missing column: Note.tags | Pass, including server restart |
| Truly absent ordinary column | Not changed | Rejected: Missing column: Note.tags |
| Existing ordinary-column lifecycle and data preservation | Not changed | Pass |

The three selected real-runtime regression tests pass. No model calls were made.
The original archived SQL and schema were compared byte for byte after replay.
An initial restricted execution could not open the local test IPC socket; the
before/after comparison and tests were run with local-server permissions enabled.

Conclusion: retain the inspector fix. It avoids one unnecessary repair in this
historical sequence. It does not fix the upstream schema's inappropriate `unique`
field, establish end-to-end correctness, or resolve the model endpoint disconnect.
