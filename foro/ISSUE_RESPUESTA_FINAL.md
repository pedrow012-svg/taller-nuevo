You are right, and I was wrong. Closing this.

I went back to my own logs and the model was never loaded by ollama at all. It was served by `llama-server` on port 8099 — my log file for that run is literally named `qwen27_llamacpp_8099.jsonl`. The `flint-40B-A2.5B-Q4_K_M.gguf` model was run through `llama.cpp`, not through this project.

So the `65,538` figure did not come from ollama, and my report mixed up two different engines. The log header I was reading said "OLLAMA" because I wrote that header myself when I started the experiment, and I never corrected it when I changed which server I was talking to. That is my mistake, not a bug in anything of yours.

**I should have checked which process was actually serving the model before filing.** I have the information in my own log files and I did not look at it. Apologies for the noise.

I am going to:

1. Delete the incorrect claim from the Reddit posts I wrote today on the back of this. One of them is already getting replies, so it will be corrected in public rather than quietly removed.
2. Take whatever the number actually means to `ggml-org/llama.cpp`, since that is where the behaviour — if it is real — belongs. I have not filed anything there yet and I will re-check my measurements before I do.

One thing I want to be precise about, because I do not want to make a second mistake while apologising for the first: I do not know yet whether `llama.cpp` silently truncates, or whether what I measured was a real truncation at all, or whether it was something in how I was configuring the server. The only thing I have established today is that **it was not ollama**, and that my report of it was wrong.

If it is any use: `ollama 0.34.1` on this machine loads every model I have tried from the library without issue, and the error message you quoted is what it says when the architecture is not one it knows. That part I can confirm.

Sorry again for taking your time with this.
