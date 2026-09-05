# research

A corpus of prior-art surveys for the [openabstractions](https://github.com/openabstractions)
project: transfer engines, IPC mechanisms, durable execution, and profiles of
tools in the local-AI space.

## Why this exists

A layered abstraction (download, storage, jobs, logging) is only worth
building where at least two real implementations already disagree in a way
a single interface can paper over — and worth stealing from where somebody
already solved the hard part. Designs in this project are meant to be
checked against prior art before they are written, not after. This
directory is that prior art: what existing systems actually do, what they
got right, what they got wrong, and what a design here would be repeating
if it ignored them.

These are surveys, not a literature review for its own sake — most were
run to answer one specific design question (e.g. "does any local tool
already expose a durable job id?") and say so at the top.

## How it is organised

Eight directories, each a topic. Read a directory's `SUMMARY.txt` first,
where one exists — it states the scope, the date, and the conclusion, and
the individual files underneath it are its sources, not summaries of their
own.

| directory | topic | start at |
|---|---|---|
| `async/` | job/task/future prior art: what survives a process exit or a reboot | `SUMMARY.txt` |
| `transfer/` | download/transfer engines (BITS, aria2, curl, HuggingFace, libtorrent, rsync/SMB, Synology) | `SUMMARY.txt` |
| `ipc/` | local IPC and channel mechanisms (Cap'n Proto, D-Bus, gRPC, varlink) | `SUMMARY.txt` |
| `integration/` | the fork surface of tools this project targets (Ollama, ComfyUI, Lemonade) | `SUMMARY.txt` |
| `data/` | one detailed technical profile per tool in the space (architecture, APIs, resource management) | any file |
| `tools/` | a second, shorter pass over most of the same tools, focused on seams and what to steal | any file |
| `logging/` | a single measurement of real log volume and placement across several installed tools | `inventory.txt` |
| `production/` | how large-scale inference providers deploy, schedule and optimise (not survey-dated the same way; treat as general background) | any file |

`data/` and `tools/` overlap in subject — the same tool often appears in
both — because they are two separate passes with different lenses: `data/`
is a broad technical inventory, `tools/` is a compact critique aimed at
what an abstraction should absorb. Neither supersedes the other.

One file, `store-interfaces-standards.md`, sits at the top level rather
than in a directory: it maps a persistence interface to external standards
and does not fit the eight topics above.

## How to read an entry

Every profile and summary states what it covers and, near the top, the
date the survey was run or the sources were last checked. Some name a
specific commit or version of the thing they describe. Look for that line
before treating anything in a file as still true.

## These are not maintained documentation

Every file here is a point-in-time survey. Software it describes moves on;
a file that says a tool "has no job id" or "does not support X" was true
when someone checked, not necessarily true now. Check the date on an entry
— and the commit or version it names, if any — before relying on it, and
prefer the primary source it cites over the summary if the two might have
drifted.

## Licence

Apache-2.0, see [LICENSE](LICENSE). Quotations, excerpts and paraphrases of
third-party documentation, source code, forum posts and other material
inside these surveys remain the property of their original authors and are
included here for review and comparison, not as this project's own work.
