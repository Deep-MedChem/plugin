# DeepMedChem plugin for ChatGPT and Claude

Search public DeepMedChem chemical spaces for molecules similar to a structure you supply, directly from **ChatGPT or Claude**. View molecular structures, similarity scores, prices and vendor catalogue IDs, then open the same saved results in [CHEESE](https://cheese.deepmedchem.com).

Both plugin packages use the same hosted MCP service and [chemical-space-search skill](skills/chemical-space-search/SKILL.md). This repository contains the package manifests, connection settings, skill and icon assets. It contains no backend server code or credentials, and using the hosted service requires no local MCP server or Python installation.

- [Set up ChatGPT](#chatgpt)
- [Set up Claude](#claude)
- [Test either platform](#try-it)
- [Build the plugin packages](#plugin-packages)

## What it does

| Tool | What it does |
|---|---|
| `catalog` — List chemical space databases | Lists the available databases, their pricing and the current guest limits. |
| `similarity_search` — Search similar molecules | Searches one SMILES against one database for up to 20 hits using `morgan` (fingerprint), `shape` or `esp` (electrostatic) similarity. |
| `render_results` — Show previous molecule results | Displays an earlier result again using its `result_id`, without submitting another search. |

In hosts that support MCP Apps, results appear as an interactive structure grid with CSV export and a link to the vendor's catalogue. The grid displays the vendor `catalog_id` when the backend supplies one. The platform `product_id` remains the API identity and is retained in CSV exports; an internal hash must not be presented as a vendor catalogue ID.

Use the **exact CHEESE link returned with the result**. A URL containing `job=` opens the saved search results without submitting another search. A general CHEESE homepage link does not identify those results. Older `run=0` links only prefill a search form and do not open a saved result.

## Set up the hosted service

Use these connection settings for either platform:

| Setting | Value |
|---|---|
| Name | DeepMedChem |
| MCP server URL | `https://mcp.deepmedchem.com/mcp` |
| Authentication | None / No authentication / No sign in |

Every user connects to the same server URL. Guest MCP access requires no DeepMedChem signup or customer API key. ChatGPT and Claude account or workspace policies still apply; CHEESE's website has its own access rules.

### ChatGPT

1. Open [ChatGPT Plugins](https://chatgpt.com/plugins) in the web version.
2. Select **+** (or **Add**) and choose **Add custom MCP server**. Some interfaces label this **Create MCP app**.
3. Enter **DeepMedChem** and the MCP server URL above. Choose **No authentication**.
4. Review the connection warning and complete creation (**Create as a plugin** in the current interface). Install the resulting plugin if prompted.
5. Start a **new chat**, type `@` and select **DeepMedChem**. If your interface uses a tools/apps menu under **+**, enable DeepMedChem there.
6. Run the [test prompt below](#try-it).

Controls and permissions vary by account and workspace. If the custom MCP option is missing, follow OpenAI's [connection and testing guide](https://developers.openai.com/plugins/deploy/connect-chatgpt) for your account; a workspace administrator may need to enable access.

### Claude

#### Claude web and desktop: remote connector

1. Open **Customize → Connectors** (shown as **Settings → Connectors** in some interfaces).
2. Choose **+ Add → Add custom connector**.
3. Enter **DeepMedChem** and the MCP server URL above.
4. Select **No sign in** if authentication is requested. No API key or request headers are needed. Finish adding the connector.
5. Start a **new chat** and enable **DeepMedChem** under **+ → Connectors**, then run the [test prompt below](#try-it).

Team and Enterprise workspaces may require an owner to add the connector before members can enable it. See Anthropic's [remote connector guide](https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp).

#### Claude Code: plugin package

Run these commands inside a Claude Code session:

```text
/plugin marketplace add Deep-MedChem/plugin
/plugin install deepmedchem@deepmedchem
```

The marketplace and plugin are both named `deepmedchem`. See the [Claude Code installation guide](https://code.claude.com/docs/en/discover-plugins) for installation scopes and workspace policies. The interactive grid depends on the host's MCP Apps support.

Connecting the MCP server directly makes its tools available. Installing the plugin package also provides the shared skill's workflow instructions.

## Try it

Use the same prompts in ChatGPT and Claude:

> Which chemical space databases can I search with DeepMedChem?

> Find 20 molecules similar to aspirin, SMILES CC(=O)Oc1ccccc1C(=O)O, in Enamine REAL using Morgan similarity, and show the results grid.

Check that:

1. A compatible host shows molecular structures, scores, prices when available and vendor catalogue IDs when supplied.
2. **CSV download** exports the displayed hits, including platform product IDs and available vendor catalogue IDs.
3. **Open in CHEESE** uses the returned `job=` link and opens the existing results. Reloading the page should show the same results without starting another search.
4. Asking **“Show me those aspirin results again”** reuses the earlier `result_id` through `render_results` instead of running another search.

After server updates, **refresh the ChatGPT connection and start a new chat**. In Claude, reconnect if the connector still shows stale tools or results, then start a new chat. Existing chat messages can retain an older grid. If your host does not display the grid, use the returned CHEESE link to view the saved results.

## Plugin packages

| Platform | Package files |
|---|---|
| ChatGPT / OpenAI | [`plugin.json`](plugin.json), [`mcp.json`](mcp.json), `skills/`, `assets/` |
| Claude | [`.claude-plugin/`](.claude-plugin/), [`.mcp.json`](.mcp.json), `skills/`, `assets/` |

To build the OpenAI upload ZIP from the current source, run from the repository root:

```sh
zip -qrX dist/deepmedchem-openai-plugin.zip plugin.json mcp.json skills assets -x '.*'
```

The output is `dist/deepmedchem-openai-plugin.zip`. Maintainer submission instructions and reviewer test cases are in [OPENAI_SUBMISSION.md](OPENAI_SUBMISSION.md). The direct connection steps above can be used to test the hosted service before directory publication.

## Limits

Guest searches accept one verified SMILES and one database at a time, with up to 20 hits. Guest capacity is shared: run searches sequentially. If the service is busy, wait briefly and retry once.

The guest tools do not resolve molecule names or support substructure constraints, property or price filters, property calculations, or SDF export. Supply a verified SMILES. For broader authenticated workflows, use [CHEESE](https://cheese.deepmedchem.com) or the [DeepMedChem Python SDK](https://github.com/Deep-MedChem/deepmedchem-python).

Similarity scores measure structural similarity, not biological activity or safety. Vendors determine availability and synthesis. Prices are catalogue list prices in USD for the default 1 mg amount shipped to the US; many make-on-demand products share the same price tier. Results are a bounded shortlist, not every match.

## Data and privacy

Both packages declare a remote MCP connection and a skill. Searches run on DeepMedChem's hosted services; the optional results grid is rendered by the assistant host.

- Tool inputs include the SMILES you search, the chosen database, method and result count, plus a `result_id` when displaying an earlier result. These tool inputs are sent to DeepMedChem; your assistant platform separately handles your conversation under its own policies.
- Requests go to `https://mcp.deepmedchem.com`, operated by Deep MedChem, which forwards searches to DeepMedChem backend services.
- Guest callers need no customer account or API key. The server's backend credential is held by DeepMedChem and is not included in either package or sent to the client. Never paste API keys into a conversation.
- Widget snapshots are kept in bounded server memory for `render_results` and CSV export. They can be evicted or lost when the server restarts, so old widget or CSV links can stop working.
- Searches routed through the CHEESE jobs API also create a stored CHEESE job containing the query and results. This is separate from the widget snapshot; restarting MCP does not delete the CHEESE job. Its handling is subject to the CHEESE privacy policy.
- Grid links open the vendor's general catalogue page or the returned CHEESE search link when clicked.

See the [privacy policy](https://cheese.deepmedchem.com/privacy-policy) and [terms of service](https://cheese.deepmedchem.com/terms-and-conditions).

## Support

Email [info@deepmedchem.com](mailto:info@deepmedchem.com), or [open an issue](https://github.com/Deep-MedChem/plugin/issues). Include the assistant platform, prompt, error and returned CHEESE link when reporting a problem.

## License

[MIT](LICENSE)
