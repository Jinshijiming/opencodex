# Plan — fix provider import criterion + verify full jawcode coverage

## Correction
The earlier import used the WRONG exclusion criterion ("CLI-agent like cursor"). The correct rule is
**protocol**: include any provider jawcode exposes on a standard streaming API opencodex already speaks
(`openai-completions`→openai-chat, `anthropic-messages`→anthropic, `openai-responses`, `azure-openai-responses`,
`google-generative-ai`); exclude only providers on a proprietary / non-HTTP-streaming protocol opencodex
has no adapter for.

## Classification (derived from jawcode models.json `api` field)
- **Include (39, standard wire):** alibaba, anthropic, azure-openai, cerebras, **cloudflare-ai-gateway**,
  deepseek, firepass, fireworks, **github-copilot**, **gitlab-duo**, google, groq, huggingface, **kilo**,
  kimi-code, litellm, minimax(+cn/code), mistral, moonshot, nanogpt, nvidia, openai, opencode(+go/zen),
  openrouter, qianfan, qwen-portal, synthetic, together, venice, vercel-ai-gateway, xai, xiaomi, zai, zenmux.
- **Exclude (6, proprietary — no opencodex adapter):** amazon-bedrock (`bedrock-converse-stream`),
  cursor (`cursor-agent`), google-antigravity + google-gemini-cli (`google-gemini-cli`),
  google-vertex (`google-vertex`), openai-codex (`openai-codex-responses` — the codex backend itself).
  (`ollama-cloud` is native `ollama-chat` in jawcode but opencodex routes it via the OpenAI `/v1` endpoint → already included.)

## Gap (vs current opencodex coverage)
Already present in opencodex (KEY_LOGIN catalog / OAUTH / init+GUI presets): all of the 39 EXCEPT four.
**ADD these 4 standard-wire providers to `src/oauth/key-providers.ts`:**

| id | opencodex adapter | baseUrl | notes |
|----|-------------------|---------|-------|
| `kilo` | openai-chat | `https://api.kilo.ai/api/gateway` | clean — all 443 models openai-completions |
| `cloudflare-ai-gateway` | anthropic | `https://gateway.ai.cloudflare.com/v1/{account-id}/{gateway}/anthropic` | clean — all 37 anthropic-messages; URL is a template |
| `github-copilot` | openai-chat | `https://api.githubcopilot.com` | **REVISED per audit** |
| `gitlab-duo` | openai-chat | `https://cloud.gitlab.com/ai/v1/proxy/openai/v1` | **REVISED per audit** |

groq/openrouter are already usable via the init+GUI presets, so they need no catalog entry.

## Audit fix (A FAILED → corrected)
The auditor proved `github-copilot` and `gitlab-duo` are NOT single-protocol — their models span
anthropic-messages + openai-responses + openai-completions. Mapping them to one `anthropic` adapter
would mis-route their GPT/Gemini models. **Fix:** map both to `openai-chat` against their universal
**OpenAI-compatible endpoint** (copilot: all models share host `api.githubcopilot.com`; gitlab: the
`/proxy/openai/v1` endpoint) — one wire that serves the whole lineup, no mis-routing.

## Auth caveat (documented, not hidden)
`github-copilot` and `gitlab-duo` are subscription gateways: they authenticate with a Bearer
subscription token (not a plain provider API key), and copilot additionally expects a client
`User-Agent` header. They're imported because the streaming IS a standard OpenAI-compatible API; the
user supplies their token (and, for copilot, may add a `User-Agent` via custom provider `headers`).
`cloudflare-ai-gateway` needs the account/gateway ids filled into the URL.

## Verify (Check phase)
A deterministic script maps EVERY jawcode provider → its api → expected include/exclude → and confirms
it against opencodex's actual catalog+presets+oauth. Pass = every standard-wire provider represented,
every proprietary one absent. Plus `tsc` clean.

## Files
- MODIFY `src/oauth/key-providers.ts` — add the 4 entries.
- (docs) MODIFY `docs-site/.../guides/providers.md` — mention the gateways/proxies + auth caveat.
