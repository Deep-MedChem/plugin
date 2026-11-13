---
name: chemical-space-search
description: Search public DeepMedChem chemical spaces for molecules similar to a supplied structure or containing required substructures, and explain the scope of the results.
---
# Chemical space search

Use the DeepMedChem MCP tools for catalog discovery, molecular similarity search and substructure search. Use similarity_search when the user only wants molecules like one structure. Use substructure_search when they fix a scaffold, a functional group or a position (see Substructure search below).

1. Call catalog for current database IDs, pricing and guest limits.
2. Use one verified SMILES per search. If only a name is supplied, resolve it using a reliable source available in the client, or ask for SMILES. Never invent a structure.
3. Choose morgan for fingerprint/Tanimoto similarity, shape for shape cosine, or esp for electrostatic cosine. Search one database at a time, at most 20 hits.
4. Preserve product IDs, returned vendor catalog IDs, SMILES, scores, metric, database and release. Platform product IDs are API identities; an internal hash is not a vendor catalog ID. Report missing prices as unavailable; do not invent currency, vendor contacts or availability.
5. In hosts that support MCP Apps, each search renders its own interactive grid of the returned structures (with scores, prices, CSV export and a link to the vendor's general catalogue page). Do not draw, generate or fetch molecule pictures by other means, and do not retype hits into a table when the grid is shown: markdown corrupts stereo SMILES such as [C@@H]. To show an earlier result again, call render_results with its result_id.
6. Explain that hits are a bounded shortlist, not all matches, a global optimum or proof of biological activity. Use the returned links.cheese URL for the search; do not replace it with a homepage or reconstruct it. A job= link opens saved results without another search; an older run=0 link only prefills the form.
7. The grid can download its displayed hits as CSV. The guest MCP does not support SDF or other file exports, property/price filters, name resolution, property calculations, UMAP, or procurement contacts. Explain missing capabilities before searching; do not substitute unconstrained similarity for requested constraints. Client-side analysis is allowed only with an available execution tool, and must be labelled as analysis of returned hits.
8. Ask for clarification when a chemical constraint is ambiguous. Do not make repeated searches to bypass guest limits. Run searches one at a time, never in parallel: guest capacity is shared and parallel calls return a busy error. On a busy error, wait briefly and retry that one search once; on repeated capacity/timeout errors, explain the failure and avoid retry loops.

## Substructure search

substructure_search returns up to 20 molecules from one database that contain every pattern in `require` and none in `exclude`. It is the CHEESE website's substructure search, written as text instead of drawn.

1. Write each pattern as one SMILES or SMARTS string without spaces. A complete scaffold can be plain SMILES (`c1ccc2[nH]ccc2c1`). Anything with query features is SMARTS and is searched exactly as written. Write charged or partial groups as SMARTS (nitro: `[N+](=O)[O-]`). In any pattern that has query features (R groups, hydrogen counts, atom lists), write aromatic rings in lowercase (`c1ccccc1`); a Kekulé ring (`C1=CC=CC=C1`) is accepted only in a plain molecule, and is rejected otherwise because it would match no aromatic ring. A pattern that cannot be read is rejected before any search runs: fix it, do not retry it unchanged. Use 1 to 6 required and at most 6 excluded patterns.
2. Call catalog first. Its `substructure.collections` lists ready-made patterns for amino acids, carboxylic acids, amines, thiols and hydrazines; prefer one of those over writing your own.
3. R group: write `[!#1]` on the atom that carries it. It means any substituent except hydrogen. Never write `R`, `[R]` or `[R1]`: these are rejected, because `[R1]` means something else in SMARTS. Example: a benzene with one open position is `c1ccc(cc1)[!#1]`.
4. Blocked position: give the atom an explicit hydrogen count so nothing can be attached there. Example: `[cH]1[cH][cH]c([!#1])[cH][cH]1` is a phenyl ring substituted at exactly one position. Atoms without a hydrogen count may carry anything.
5. Separate patterns are independent: each must be present somewhere, in any arrangement. To fix how two groups are connected, write them as one pattern.
6. The patterns filter; the reference only ranks. `reference_smiles` is an optional complete molecule (no R groups or wildcards) and `method` (morgan, shape or esp) orders the matching hits by similarity to it. Without a reference the hits are unranked and have no score; never describe them as "most similar".
7. Report what the search covered. When `search_scope.outcome.message` is present, repeat it, with `coverage` when given. Never say that no matching molecule exists unless `search_complete` is true. A timed-out or incomplete search with few or no hits means "not found in the part searched". An unranked search normally reports "Search is incomplete" because a small unindexed part of the library is never searched; state it once as the scope of the result, not as an error.
8. A search with several large patterns can take up to a minute and can time out with partial hits. Say so; do not silently retry with looser patterns. If the user agrees to loosen a pattern, say which one changed.
9. Not available here: flexible linkers, variable ring sizes and core marks (these exist only in the CHEESE drawing editor), and the dedicated amino-acid building-block databases (search those on the CHEESE website). A Markush claim with long substituent lists is searched by its core with open positions; say that the substituent lists were not enforced.
10. The CHEESE page opened by links.cheese shows the first required pattern as the query. The full list of conditions is in `search_scope.constraints_applied`; quote it when the user asks what was searched.

Worked examples (arguments as run against Chemspace Freedom Space):

- Molecular glue, "keep these two scaffolds": `require: ["O=C1CCC(N2C(=O)c3ccccc3C2=O)C(=O)N1", "c1ccc2[nH]ncc2c1"]`. Two whole scaffolds is the slowest kind of query and often returns partial hits with a timed-out message.
- Covalent inhibitor, "fix the carboxylic acid on the ring and vary the position opposite it": `require: ["[OH1,O-]C(=[OX1])c1[cH][cH]c([!#1])[cH][cH]1"]`, `reference_smiles: "OC(=O)c1ccc(cc1)-c1ccncc1"`, `method: "shape"`. One connected pattern fixes where the acid sits; `[cH]` blocks the other ring positions. Two separate patterns (an acid, a substituted benzene) would only require both groups somewhere in the molecule.
- Patent Markush, "para-substituted N-substituted benzamides, no nitro": `require: ["[cH]1[cH]c([!#1])[cH][cH]c1C(=O)N[!#1]"]`, `exclude: ["[N+](=O)[O-]"]`.
- Peptide optimisation, "non-canonical amino acids to replace phenylalanine": `require: ["[NX3H2,NX4H3+][CX4]([!#1])C(=[OX1])[OH1,O-]"]` (the catalog's α-amino acids pattern), `reference_smiles: "N[C@@H](Cc1ccccc1)C(=O)O"`, `method: "morgan"`. The hits are free amino acids ranked by similarity to the residue, not whole peptides.

For broader authenticated workflows, direct the user to https://cheese.deepmedchem.com and the DeepMedChem Python SDK. Never ask users to paste API keys into chat.
