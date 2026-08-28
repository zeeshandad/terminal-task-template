# Proposal: Corrupted SQLite Database Recovery and Forensic Reconstruction

**Author:** Zeeshan Dad Khan

## What the task is
The environment contains a partially corrupted, undocumented SQLite database file alongside a markdown instruction written in the "realistic brief" style. The instruction acts as a ticket from a staff engineer detailing strict business rules for data integrity. The solver must perform a forensic recovery: extracting the surviving valid records, reconstructing broken foreign key relationships by cross-referencing surviving tables, and outputting a byte-exact, canonical JSON array of the recovered transaction data. The environment is strictly offline (`network_mode = "no-network"`) and all dependencies will be vendored at build time. 

## Why it's intrinsically hard for an agent
This task is intentionally adversarial and designed to target the ~20% pass rate acceptance band for frontier models like `google/gemini-3.6-flash`. The difficulty captures a specific gap between developers and agents: the agent's reluctance to perform forced investigation. Standard automated recovery commands (like `sqlite3 .dump` or standard Python/C# library repairs) will silently drop rows that violate corrupted constraints. The agent cannot rely on documentation; it must manually inspect the schema discrepancies via the environment to understand why standard queries fail. A single wrong assumption during the extraction phase creates compounding state failures, resulting in a total failure of the precise behavioural contract at the end.

## Planned Traps
As required, this task includes two specific traps for plausible-but-wrong approaches:
1. **The Custom Collation Trap:** The database utilizes a custom collation sequence on a critical `UNIQUE` constraint. When an agent attempts a standard data extraction, standard tools default to stripping unrecognized collations. This looks like a successful recovery on the surface but silently alters the output sorting order, guaranteeing failure on the byte-exact output contract.
2. **The Embedded Null-Byte Trap (Nasty-but-fair edge):** A specific text field contains intentional embedded null bytes (`\x00`) used as legacy delimiters, which constitutes a nasty-but-fair edge case. Naive string parsing functions will prematurely terminate upon hitting the null byte. This looks completely correct during naive checks but will fail a hidden fixture specifically designed to catch the truncated strings.

## Expected human solve time
45 to 60 minutes. The instruction will be highly legible, allowing a principal engineer to read it in two minutes and know exactly what "done" means. A skilled human will recognize that automated tools are failing, use basic SQL inspection commands to spot the custom collation/delimiters, and write a targeted script to extract the data precisely. 

## Asset Origins
All assets for this task will be 100% original. The initial database schema, the script used to intentionally corrupt the file during the image build, the reference solution, and the validation tests will be authored entirely by me. No data, structures, or code will be adapted from public benchmarks, tutorials, well-known open-source software, or any employer's code. A complete `provenance.json` will be included in the final delivery.