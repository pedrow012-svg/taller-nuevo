Following up with measurements, because @rick-github was right and I was wrong — but the fix turns out to be narrower than the release note suggests.

I installed `0.40.16` side by side with `0.34.1` and ran both against the same document, same model (`qwen2.5:3b`), same machine, through `/api/chat`:

| version | `num_ctx` | `prompt_eval_count` | warned | what it kept |
|---|---|---|---|---|
| `0.34.1` | default | **16,386** | **no** | the tail |
| `0.34.1` | `32,768` | **16,386** | **no** | the tail |
| `0.40.16` | default | **16,386** | **no** | the tail |
| `0.40.16` | `32,768` | **16,386** | **no** | the tail |

**Through the native endpoint, `0.40.16` behaves exactly like `0.34.1`.** Same number, no error, no warning, `done_reason` not signalling anything useful. Doubling the input to 600,000 characters does not move the count by a single token.

Through `/v1/responses` the difference is night and day, and this part confirms your point:
- `0.34.1`: `16,386` read, `status: completed`, no error, no content. Silence with a success label.
- `0.40.16`: `request (94292 tokens) exceeds the available context size (32768 tokens), try increasing it`.

So the release note is accurate — `/v1/responses` respects `truncation` now — but clients using `/api/chat` get no benefit from upgrading.

Two more findings that may explain other reports:

1. **The prompt budget is half the window.** `num_ctx=4096` reads `2,050`. `num_ctx=32768` reads `16,386`. `num_ctx=65536` reads a document that fits, in full (`31,478`). Systematic, not a memory limit.
2. **The truncation drops the head, not the tail.** A document with a unique word on its first line and another on its last: asked about the first, the model answers filler and misses it; asked about the last, it answers correctly.

If `/api/chat` is meant to truncate silently at half the window while `/v1/responses` now errors, that is worth documenting at minimum — I spent two days assuming my instrument was broken rather than my server, and I was half right about that too.

I am not filing this as a request, just leaving the numbers where the next person will find them. If the native-endpoint behaviour is intentional, tell me and I will write it up as such.
