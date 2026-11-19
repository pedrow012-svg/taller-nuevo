### What happens

I send a long document to the API. The model reports reading **65,538 tokens**, no matter how much I send. And `done_reason` comes back `stop` — not `length` — so nothing indicates a truncation happened.

Two runs, documents of different sizes, identical token count:

| sent | tokens reported as read |
|---|---|
| 300,000 characters | **65,538** |
| 600,000 characters | **65,538** |

Doubling the input does not move the number. It is a hard ceiling.

### Why I think this is not the configured context window

The context window in use was **147,456 tokens** — read from the running process, not from what I passed in. The model read **65,538**, which is **less than half** of the available window.

So the cut happens below the model's own limit. It is not the model, and not the KV cache size.

### What I expected

Either the full document to be processed, or an explicit signal that it was not — an error, a warning, `done_reason: length`, anything.

### What I actually got

A clean `200` response, `done_reason: stop`, and an answer that confidently referred to content near the start of the document while fabricating content near the end.

### Reproduction

1. Build a document of more than 100,000 tokens.
2. Place a unique, unmistakable sentence **in the very last line** (e.g. "the secret word is MARMOLINA").
3. Send it through `/api/chat` and ask for that fact.
4. Read `prompt_eval_count` (or `usage`) from the response.

Expected: the model finds it, or reports it could not read everything.
Actual: the token count comes back pinned at **65,538**, and the model answers as if the document ended earlier.

The number to watch is not the answer — it is `prompt_eval_count`. It is the same on every run.

### Environment

```
ollama version : 0.34.1
model          : flint-40B-A2.5B-Q4_K_M.gguf
context window : 147,456 (read from the running process)
GPU            : 2× 16 GB (one model split across both)
system RAM     : 62 GB
OS             : Linux
OLLAMA_* env   : none set, everything at defaults
```

### Numbers from one run

```
input          : 600,000 characters
prompt tokens  : 65,538   (114 s)
generated      : 370 tokens
done_reason    : stop
```

### What I have not tested

I want to be precise about the limits of this report:

- **I have not tested other ollama versions.** Only 0.34.1.
- **I have not tested raising `num_ctx` beyond 147,456** to see whether the 65,536 ceiling moves. If it does, this is a configuration issue and I will say so.
- **I have not tested other models.** Only the one above. But the ceiling being identical at two input sizes suggests it is not model-specific.
- **I have not tested llama.cpp directly** to see whether the same input hits the same wall.
- **No `OLLAMA_*` environment variables were set.** Everything is at defaults.

If any of those would change the result, I am happy to run them — just tell me which one matters.

### Why I am filing this

I use ollama for long-document work. If documents are being cut silently and the response says `stop`, then answers are being built on text the model never saw, and nothing in the response says so.

I would rather be told "I could not read all of it" than be told a confident answer.
