# Experiment 02: clarify composite constraints in the schema prompt

Status: inconclusive; candidate production edit withdrawn.

Hypothesis: explicit instructions to represent multi-field rules as constraints on
business fields, rather than additional marker columns, would prevent the schema
from inventing a `RecentlyViewed.unique` column.

Reused the exact request that originally produced the marker field: BookStack run
`bookstack-local-resumed-20261107-205034`, call
`346219ee5d3c4c47a951e633d6c77508` (REQ-7.1 schema context query).
No earlier stage was regenerated. The candidate added only these prompt lines:

> Represent a rule involving several fields, such as uniqueness of an identity tuple,
> as a constraint description on the participating business fields. A constraint is
> not an additional stored value: do not invent a marker column to hold the rule.

All user messages, context, model and generation parameters were identical.
The user explicitly authorized two paid requests to the existing ARC Bench endpoint
using existing local credentials. Both variants used no SDK retries and a 90-second
timeout, to bound this experiment. No credentials were written to the evidence.

| Variant | Elapsed | Raw failure | Model response |
| --- | ---: | --- | --- |
| Original prompt | 37.36 s | RemoteProtocolError: Server disconnected without sending a response. | None |
| Candidate prompt | 1.97 s | ConnectError: SSL UNEXPECTED_EOF_WHILE_READING | None |

The first failed while waiting for an HTTP response; the second failed in TLS.
Neither is evidence for or against the schema-prompt hypothesis. Failed requests
have no reported token usage; do not interpret this as zero usage or zero charge.
No additional retries or full-stage runs were started.

Two existing schema merge/reference-validation tests pass locally, but they do not
establish whether a live model would follow the candidate prompt. The three-line
candidate change was removed so only the independently verified inspector fix
remains in production. The upstream schema semantics and endpoint reliability
remain unresolved.

Detailed local requests and exception chains are retained in
`runs/experiment-schema-constraints-20261107/` in the enclosing workspace; they
are not part of this repository. Future work should first establish a usable
request path with a bounded diagnostic before spending on dependent stages.
