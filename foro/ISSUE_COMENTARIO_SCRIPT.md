Here is the reproduction script. No dependencies beyond `python3` and a running ollama — it builds the documents itself, so there is no file to download.

```python
import argparse, json, sys, time, urllib.request

SECRETO = "MARMOLINA"

def construir_documento(n_chars, relleno):
    cuerpo = (relleno + "\n") * (n_chars // (len(relleno) + 1) + 1)
    cuerpo = cuerpo[:n_chars]
    return cuerpo + "\nLA PALABRA SECRETA ES " + SECRETO + ".\n"

def preguntar(host, modelo, texto, timeout=1800):
    cuerpo = {
        "model": modelo,
        "messages": [{"role": "user", "content":
            texto + "\n\nPregunta: ¿cuál es la palabra secreta del documento? Responde sólo la palabra."}],
        "stream": False,
        "options": {"temperature": 0},
    }
    req = urllib.request.Request(host.rstrip("/") + "/api/chat",
                                 data=json.dumps(cuerpo).encode(),
                                 headers={"Content-Type": "application/json"})
    t0 = time.time()
    r = urllib.request.urlopen(req, timeout=timeout)
    d = json.loads(r.read())
    msg = d.get("message", {}) or {}
    return ((msg.get("content") or msg.get("thinking") or "").strip(),
            d.get("prompt_eval_count"), d.get("done_reason"), time.time() - t0)

host, modelo = "http://127.0.0.1:11434", "YOUR_MODEL_HERE"
relleno = "El taller mide, apunta y no inventa. " * 20

for n in (300_000, 600_000):
    doc = construir_documento(n, relleno)
    resp, leidos, motivo, dt = preguntar(host, modelo, doc)
    print(f"{n:>7,} chars -> leidos {leidos} tokens · done_reason={motivo} · {dt:.0f}s")
    print(f"         encontro la palabra? {'SI' if SECRETO.lower() in resp.lower() else 'NO'}  · dijo: {resp[:60]!r}")
```

**What to look at**, in order of importance:

1. **`prompt_eval_count`** — the tokens actually read. **This is the number that matters**, not the answer.
2. **Whether it is the same on both runs** despite 2× the input.
3. **`done_reason`** — if it says `stop` while the document was cut, that is the bug.

On my machine the two numbers came back identical (`65,538`) and the secret word at the end was never found, with `done_reason: stop`.

**Sanity check, so you know the script is not just failing at everything:** with a short document (5,000 characters) the same script finds the secret word immediately and `prompt_eval_count` scales normally (1,688 → 3,314 tokens when I doubled the input). It only flatlines at the ceiling.

If someone runs it and gets a *different* number, please post it — that would mean the ceiling depends on something I have not identified, and that is worth knowing too.
