# OpenAI plugin submission material

Open https://platform.openai.com/plugins and upload the **0.1.1 OpenAI ZIP** to the
existing DeepMedChem plugin. Use **Upload new or existing plugin / Upload plugin**;
do not create a second listing. Version 0.1.1 includes the support URL, the valid
category, five positive and three negative review cases, and release notes.

Use the ZIP attached to the matching GitHub release, or build it from a clean
checkout of that release:

```sh
mkdir -p dist
zip -qrX dist/deepmedchem-openai-plugin.zip plugin.json mcp.json skills assets -x '.*'
```

The OpenAI package uses `plugin.json`, `mcp.json`, `skills/` and `assets/`. The Claude
plugin uses `.claude-plugin/` and `.mcp.json`. Both share the same skill.

## Before submitting

1. Select **0.1.1 · Draft** and wait for metadata and skill checks to finish.
2. Under **MCPs**, open the DeepMedChem server and check that its URL is
   `https://mcp.deepmedchem.com/mcp` and authentication is **No Auth**.
3. Leave **Challenge Base URL** blank to use the MCP hostname. The exact portal
   token must be served as plain text at
   https://mcp.deepmedchem.com/.well-known/openai-apps-challenge. The operator sets
   this on the hosted service; it is not included in the public ZIP. Click
   **Verify Domain**, then **Connect**, and wait for the tool scan.
4. If a privacy-policy accessibility check offers **Retry**, retry it and inspect
   any remaining finding. The linked policy must be accessible to reviewers.
5. Refresh the ChatGPT test connection and start a new chat. Run the cases below;
   verify the grid, CSV export and saved CHEESE links.
6. Open **Add review information**. Check the imported cases and release notes,
   add the reviewer-accessible demo video URL, and save. Reviewer access is
   unauthenticated; no test account or customer API key is needed.
7. Choose **Submit for review** once required findings are resolved. The owner
   completes the portal's policy attestations. Public directory publication is
   a separate action after approval.

See OpenAI's [submission guide](https://developers.openai.com/plugins/deploy/submission)
and [submission error reference](https://developers.openai.com/plugins/deploy/submission-errors).

## Listing

| Field | Value |
|---|---|
| Display name (≤30) | DeepMedChem |
| Short description (≤30) | Molecular similarity search |
| Category | Education & Research |
| Long description | from `plugin.json` `longDescription` |
| Icon | the 256×256 PNG icon in the assets folder |
| Website URL | https://deepmedchem.com |
| Support URL | https://github.com/Deep-MedChem/plugin/issues |
| Privacy policy URL | https://cheese.deepmedchem.com/privacy-policy |
| Terms of service URL | https://cheese.deepmedchem.com/terms-and-conditions |
| MCP server | `https://mcp.deepmedchem.com/mcp`, authentication None |

Reviewer access: public, read-only endpoint. No account, login or test credentials.

## Five positive test cases

The authoritative prompts, expected tools and expected behavior are declared in
`plugin.json` under `extensions.com.openai.review.test_cases`. They are imported
with the ZIP. To change imported cases, edit the package and upload a new version.

1. **List databases.** Prompt: "Which chemical space databases can I search with
   DeepMedChem?" Tool: `catalog`. Expected: the databases, identifiers, methods,
   pricing and guest limits returned by the current catalog.
2. **Fingerprint search.** Prompt: "Find 20 molecules similar to aspirin, SMILES
   CC(=O)Oc1ccccc1C(=O)O, in Enamine REAL using Morgan similarity, and show the
   results grid." Tools: `catalog`, `similarity_search` (method `morgan`).
   Expected: up to 20 hits ranked by fingerprint similarity, with scores,
   available prices and vendor catalog IDs when supplied. Also check CSV export
   and the returned saved CHEESE link as described below.
3. **Shape search.** Prompt: "Search Enamine REAL for shape-similar analogues of
   c1ccc(CCNc2ncnc3ccccc23)cc1." Tool: `similarity_search` (method `shape`).
   Expected: grid of up to 20 hits ranked by shape cosine.
4. **Electrostatic search.** Prompt: "Search Enamine REAL for up to 20 molecules
   electrostatically similar to CC(=O)Oc1ccccc1C(=O)O using ESP similarity and show
   the grid." Tools: `catalog`, `similarity_search` (method `esp`). Expected: up to
   20 returned structures with the electrostatic similarity metric.
5. **Show an earlier result again.** After case 2, prompt: "Show me those aspirin
   results again." Tool: `render_results` with the earlier `result_id`. Expected: the
   same grid, same structures and scores, without a new search.
### Additional checks within the fingerprint case

- **CSV export.** In the grid from case 2, click the CSV download. Expected: a CSV
   with the same SMILES, platform product IDs, scores and prices as the grid,
   plus the returned vendor catalog IDs when available. Missing IDs remain blank.
- **Open saved CHEESE results.** In a fresh grid from the updated production
   server, click its CHEESE link. Expected: a `job=` URL opens the existing
   results, without another Search action or a new paid search. Refresh that
   page and check the same stored results remain visible. A `run=0` link means
   the older server is still being served and this release is not ready.

Complete these checks in both ChatGPT and Claude before submitting. Verify that
the server's CHEESE API credential is accepted and its service-account allowance
supports the published guest capacity. Never upload that credential with either
plugin package.

## Three negative test cases

1. **Name without a structure.** Prompt: "Find molecules similar to cetirizine."
   Expected: the assistant does not invent a structure. It asks for a SMILES, or uses
   a reliable source it has, and says DeepMedChem itself does not resolve names.
2. **Unsupported constraint.** Prompt: "Find analogues of CC(=O)Oc1ccccc1C(=O)O with
   logP below 1 and price under $50." Expected: the assistant explains that property
   and price filters are not supported before searching, and does not present an
   unfiltered similarity search as satisfying the constraint.
3. **Beyond the guest cap.** Prompt: "Give me 500 molecules similar to aspirin."
   Expected: the assistant explains the 20-result guest limit, returns at most 20,
   and does not loop searches to get around it.

## Release notes

Initial directory submission of DeepMedChem for ChatGPT: guest molecular
similarity search using Morgan, shape and ESP methods, with a results grid, CSV
export and saved CHEESE links. Package 0.1.1 corrects the support URL and category
and includes five positive and three negative review cases.

## Demo recording checklist

Record the ChatGPT web interface with a fresh DeepMedChem connection:

1. Show the enabled plugin, list databases, and run the aspirin Morgan prompt.
2. Show the molecular grid and catalogue IDs. Download the CSV and open the
   returned CHEESE link; reload it to demonstrate the same stored results.
3. Return to the chat and ask to show the aspirin results again.
4. Run the shape and ESP prompts, one at a time, and show their grids and metrics.
5. Run the three negative prompts and show the clarifications or limitations.

Upload the recording to a URL reviewers can open without requesting access.
Open the link in a signed-out browser to check access before pasting it into
**Add review information**. Avoid showing account settings, credentials or private
customer structures during the recording.

## Still needed

- Video walkthrough of the cases above, at a reviewer-accessible URL.
- Organization-level access: a verified company organization, and Owner or Apps
  Management Write for the submitter.
