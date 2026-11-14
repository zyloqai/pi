What are we learning with Pi?

**How can the same model give a better answer when we change the software around it?** In these five labs, we use Pi to investigate that question inside the AI Systems Performance Observatory. We keep Kimi fixed, change one part of the harness at a time, and check the result against something we can verify ourselves.

## Start with the 720 ms question

When you asked which stage took longest for request A, where did Kimi get **720 ms**?

1. **Pi** sent your question and the definition of `get_timing` to **Kimi** through OpenRouter. The definition offered two IDs, A and B.
2. **Kimi** requested `get_timing` with `{"request_id":"A"}`. **Pi** checked the argument and ran your tool on the Ubuntu VM.
3. **Your tool** chose the filename and started `python3 timing.py A`. Python printed the example record. Pi sent that result back to Kimi.
4. **Kimi** compared the stages and answered “queue, 720 ms.” When you later asked for G, Kimi read the same A/B rule and answered directly. There was no tool call on that turn.

That software surrounding the model is the **harness**. It decides what information and actions the model receives, runs the actions it allows, and carries results into the next turn. In this experiment, the model’s weights stayed fixed. The 720 ms came from our synthetic record. [Replay Project 1](01-trace-tool/project-1-field-guide.html)

## What each project adds

| Project | What we change in Pi | How we know whether it helped |
|---|---|---|
| [1. Give Kimi a timing tool](01-trace-tool/README.md) | Replace a general Bash command with a tool that accepts one request ID. | Inspect the call. Who chose the ID? Who chose the filename? Where did the number first appear? |
| [2. Choose a batch size](02-batching/README.md) | Give Kimi a reusable prompt with a first-token latency limit. | Calculate which rows qualify and which qualifying row delivers the most tokens per second. |
| [3. Estimate KV-cache memory](03-kv-cache/README.md) | Give Kimi a reusable calculation procedure, called a Pi skill. | Derive bytes per token by hand and check the memory estimate as the sequence grows. |
| [4. Repair a latency metric](04-metric-repair/README.md) | Give Kimi a failing test as feedback on a time-to-first-token calculation. | Explain which two timestamps belong in the calculation, make the edit, and rerun the tests. |
| [5. Evaluate a prompt change](05-harness-eval/README.md) | Change one instruction and give the same model the same six traces. | Score both sets of answers and inspect a case that explains a score or a tie. |

## Where we are now

**Project 1’s tool loop has been observed on your VM. Project 2 is next.** In Project 1, *batching* meant sending several IDs in one tool call. In Project 2, *batch size* is an inference-server setting that limits how many requests the server processes together. The four rows are synthetic benchmark runs.

**First prediction for Project 2:** Batch 8 delivers 24,000 output tokens in 120 seconds and has a p95 time to first token of exactly 250 ms. If the allowed limit drops from **250 ms to 249 ms**, can you still choose batch 8? Which batch would you choose instead?

Open the data and make your choice before asking Pi. Calculate throughput as **output tokens ÷ elapsed seconds**. First filter by the inclusive latency limit, then rank the qualifying rows by throughput. When you enter `/compare-batches 250`, Pi inserts 250 into the prompt template and Kimi uses the file to answer the same question. [Run data](02-batching/runs.csv) · [Prompt template](02-batching/.pi/prompts/compare-batches.md)

After all five labs, we’ll draw Pi’s architecture from memory and use these runs to decide which parts to rebuild and which limitations are worth testing.
