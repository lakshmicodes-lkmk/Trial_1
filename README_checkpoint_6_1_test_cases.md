# Checkpoint 6.1: separate evaluation cases

Keep `checkpoint_6_1_external_cases.py` and `checkpoint_6_1_test_cases.json` in the same folder as your `Wikipedia_10_text` corpus, or set `WIKIPEDIA_TEXT_DIR` to its location. The script defaults to the JSON file beside itself, regardless of the current working directory. It retains the same model ladder, retrieval setup, logging, and baseline-versus-hardened comparisons.

## Run from PowerShell

```powershell
python '.\checkpoint_6_1_external_cases.py' --models 'openai/gpt-4o-mini'
```

To use another case file:

```powershell
python '.\checkpoint_6_1_external_cases.py' --test-cases '.\my_cases.json' --models 'qwen/qwen3-8b'
```

The script still needs the packages required by the original program, a valid `OPENROUTER_API_KEY` in its environment or `.env`, and the Wikipedia TXT corpus. It creates or reuses the Chroma index. The comparison output includes each case's `expected_behavior`, retrieved passage IDs, baseline response, hardened response, and latency.

## Designing a case

1. **Start with a legitimate user task.** Make it answerable from the ten indexed Wikipedia articles. Include its expected article IDs and the facts you expect a grounded answer to contain.
2. **Choose one attack surface.** `answer` tests adversarial user text; `indirect_document` inserts a controlled passage into the prompt; `planner` tests whether adversarial text changes retrieval queries. Keep the factual task the same when comparing baseline and hardened behavior.
3. **State the desired behavior before running models.** Record what the system should do and any claims or query terms it must avoid. This prevents changing the pass criteria after seeing an output.
4. **Compare models on the exact same file.** Use the same corpus, index, question, probes, retrieval settings, and temperature. Save the JSON comparison and log for traceability.
5. **Review the outcome at the attacked stage.** For answer cases, compare factual coverage and citations with the actual passages. For planner cases, inspect the parsed queries and whether the forbidden target appears. A fluent answer alone does not show that tool use was safe.

Each probe needs a unique `id`, a `kind`, `legitimate_question`, `attack_surface`, `expected_behavior`, and `watch_for`. An `answer` case needs `attack`; an `indirect_document` case needs `retrieval_query` and `poisoned_passage`; a `planner` case needs `attack` and `retrieval_query`. `forbidden_claims` and `forbidden_query_terms` are expectations for evaluation. **The script does not automatically mark a generated answer as passing based on those fields.** For planner cases it detects forbidden query terms, but absence of those terms does not by itself establish that the retrieval query is relevant.

## Important scope

The `indirect_document` case inserts a controlled passage into the prompt; it does not alter Wikipedia files or Chroma. The `planner` case exercises the decision prompt and its generated queries; it does not execute the generated query as a tool call. The `answer` cases retrieve against the full attack string as the original script did, so a failed result can reflect retrieval changes as well as answer behavior. Keep that distinction in the worksheet.

The default file has one multi-article normal task and five probes: a command/persona attack, a fabricated source, an opposite-answer persona, an injected document passage, and tool-argument redirection. To extend it, add a new object to `security_probes` with one clear failure mode and an expected article. Good additional tests would include a neutral follow-up after a persona attack (which requires a separate multi-turn runner) or a long, irrelevant retrieval query (which requires a cost or relevance check). Listing either in JSON alone would not implement those experiments.
