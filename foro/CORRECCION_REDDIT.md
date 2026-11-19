CORRECTION — I was wrong about this, and I am leaving it up rather than deleting it.

When I filed this I stated the truncation was ollama's. It was not. I went back to my own logs: the model in that run was served by `llama-server` on port 8099, not by ollama. My results file for that run is literally named `qwen27_llamacpp_8099.jsonl`. The log header said "OLLAMA" because I wrote that header myself and never corrected it when I switched engines.

**So the 65,538 figure did not come from ollama.** I have since confirmed that ollama 0.34.1 refuses to load that model at all ("unknown model architecture"), which settles it — it could never have produced those numbers.

I filed it on github.com/ollama/ollama (#19765) and the maintainers caught it immediately. I have apologised there and closed it.

**What I still don't know**, and won't claim: whether `llama.cpp` really does truncate silently at that number, or whether what I measured was something in my own server configuration. I am not confident enough in my own instrumentation today to file that anywhere.

The lesson is duller than the finding was: **check which process is actually serving the model before you write up what it did.** I had the evidence in my own log files and did not look at it.

Sorry for the noise.
