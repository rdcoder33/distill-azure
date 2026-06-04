# distill

Agent command outputs are one of the biggest sources of token waste.

Logs, test results, stack traces… thousands of tokens sent to an LLM just to answer a simple question.

**🔥 `distill` compresses command outputs into only what the LLM actually needs.**

Save **up to 99% of tokens** without losing the signal.

## How to use

Install:

```bash
npm i -g @samuelfaj/distill
```

Run onboarding:

```bash
distill
```

And that's it!

## Using this fork locally

This fork adds Azure OpenAI / Microsoft Foundry request shaping for newer
OpenAI-compatible deployments. The npm package name is still
`@samuelfaj/distill`, so use a local link while developing or testing the fork.

From this repo:

```bash
npm install
npm run build
npm link ./packages/cli
hash -r
```

Verify the linked CLI is the fork:

```bash
which distill
realpath "$(which distill)"
distill --version
```

`realpath "$(which distill)"` should point inside this checkout, for example:

```text
/path/to/distill-azure/packages/cli/bin/distill.js
```

Do not install this fork directly with
`npm install -g github:rdcoder33/distill-azure` while it remains a workspace.
The root package is not the CLI package, and native platform binaries are
generated build artifacts.

To return to the published upstream package:

```bash
npm unlink -g @samuelfaj/distill
npm i -g @samuelfaj/distill
hash -r
```

## Azure OpenAI / Microsoft Foundry

Configure the fork as an external OpenAI-compatible provider:

```bash
distill config provider external
distill config host "https://<resource>.openai.azure.com/openai/v1"
distill config model "<azure-deployment-name>"
distill config api-key "<azure-api-key>"
```

Foundry-style hosts are also detected when the hostname ends with:

```text
.openai.azure.com
.cognitiveservices.azure.com
.services.ai.azure.com
```

For Azure/Foundry hosts, deployments whose model name starts with `gpt-5`,
`o1`, `o3`, or `o4` send `max_completion_tokens` instead of `max_tokens`.
Older models keep `max_tokens`.

Run a smoke test with dataset capture disabled:

```bash
printf 'line 1: keep\nline 2: discard\nline 3: keep\n' \
  | DISTILL_DATASET_ENABLED=false distill "Return only lines containing keep."
```

Expected:

```text
line 1: keep
line 3: keep
```

Do not commit API keys or real Azure resource names. Keep
`~/.config/distill/config.json` private and mode `600`.

## Our Model

Distill uses it's own **Expert Language Model** 

https://huggingface.co/samuelfaj/distill-1.7B-MLX

**Only: 1.7B - 4bit**

Safe recommendation: machine should have 8 GB+ RAM; 16 GB+ is comfortable.

## Example

```sh
rg -n "terminal|PERMISSION|permission|Permissions|Plan|full access|default" desktop --glob '!**/node_modules/**' | distill "find where terminal and permission UI are implemented in chat screen"
```

- **Before:** [7648 tokens 30592 characters 10218 words](./examples/1/BEFORE.md)
- **After:** [99 tokens 396 characters 57 words](./examples/1/AFTER.md)
- **🔥 Saved ~98.7% tokens**

## Distill Language

We also teach your LLM to talk and think a more efficient way.

![Distill Language](https://github.com/samuelfaj/distill/blob/main/examples/distill-language.png?raw=true)

### Label legend

When `/distill` is active, responses use readable labels instead of one-letter
prefixes:

```text
Status: current state
Context: why this matters
Action: what will happen or what happened
Risk: blocker or failure mode
Outcome: result
Constraint: no-go or limit
Proof: verification or pass criteria
```

Example:

```text
Status: tests running
Context: Azure GPT-5 rejects max_tokens
Action: patch request body
Risk: wrong host suffix may miss detection
Outcome: tests pass
Constraint: no API keys in docs
Proof: npm run test PASS
```
