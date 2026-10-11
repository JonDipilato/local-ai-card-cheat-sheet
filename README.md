# Which file for your card: 8, 12, 16, 24 (and 32) GB

What runs on which card, from my own tests on an RTX 4070 SUPER 12GB with 32GB of RAM. Each row is marked one of two ways:

- **TESTED**: I ran it on my PC and these are my numbers.
- **FITS BY SIZE**: worked out from the file size on Hugging Face. **I have not run it on that card.** Treat it as a starting point.

## The rule I use

On my 12GB card, the 8.2 GB file used about **11.5 GB** once loaded with 8K context. So the file plus about **3.3 GB** for context and Windows. Pick a file at least ~3.5 GB smaller than your card. More context takes more room.

The one setting that matters most: Ollama loaded this model with a 262,144-token context by default, which ate about 19 GB extra and made even the 11 GB file crawl at under 3 tok/s. Set it to 8K:

```
ollama run <model>
/set parameter num_ctx 8192
```

Same file, same card, about 4x faster.

## Qwen 3.8 27B: the right file per card

Files are from [unsloth/Qwen3.8-27B-GGUF](https://huggingface.co/unsloth/Qwen3.8-27B-GGUF). Pull any of them with:

```
ollama pull hf.co/unsloth/Qwen3.8-27B-GGUF:<QUANT>
```

| Your card | File to use | Size on Hugging Face | Status |
|---|---|---|---|
| **8 GB** | None. The smallest 27B file (UD-IQ1_S, 6.19 GB) leaves no room for context. Use a smaller model, see below. | | Doesn't fit, by size |
| **12 GB** | **UD-IQ2_XXS** | 7.27 GB (8.2 GB in Ollama, which adds the vision part) | **TESTED**: 100% on the card, 42 to 48 tok/s at 8K context |
| **16 GB** | **UD-IQ3_S** (UD-Q3_K_XL at 13.15 GB if you keep context small) | 12.04 GB | FITS BY SIZE |
| **24 GB** | **UD-Q5_K_M** (UD-Q4_K_XL at 17.56 GB for more context room) | 19.77 GB | FITS BY SIZE |
| **32 GB** | **UD-Q6_K_XL** | 25.30 GB | FITS BY SIZE |

### What happened when the file didn't fit my 12 GB card (TESTED, August 2026, 8K context)

| File | Split card / RAM | Speed |
|---|---|---|
| UD-IQ2_XXS tag (9.5 GB file at the time*) | 100% card | 42.8 tok/s |
| UD-Q2_K_XL (11 GB) | 82% / 18% | 11.5 tok/s |
| UD-IQ3_XXS (12 GB) | 74% / 26% | 5.4 tok/s |
| Ollama default `qwen3.8:27b` (18 GB) | 47% / 53% | 4.7 tok/s |

\*Unsloth has since re-uploaded the file behind that tag. Today it downloads the 8.2 GB file, which I re-tested in September at 46 to 48 tok/s.

Even 18% spilling into RAM cut the speed by about 4x. Fit the whole file on the card.

**The catch on 12 GB:** the IQ2_XXS file is a 2-bit squeeze. In my 6-job test it passed 3, tied 1 and failed 2 (it kept rewriting the invoice file and contradicted a facts list). Fine for quick texts and small pages, weaker on long or careful work. Full test kit: [27b-vs-flash-next-test-kit](https://github.com/JonDipilato/27b-vs-flash-next-test-kit).

## Smaller models I've run on the same 12 GB card

| Model | Size in Ollama | Status |
|---|---|---|
| `qwen2.5-coder:7b` | 4.7 GB | **TESTED**: 73.83 tok/s. This is the one for **8 GB cards**, FITS BY SIZE on 8 GB |
| `qwen2.5-coder:14b` | 9.0 GB | **TESTED**: fits fully on 12 GB |
| `qwen2.5-coder:32b` | 20 GB | **TESTED**: spilled into RAM on 12 GB and crawled. FITS BY SIZE on a 24 GB card |

## 12 GB card + 32 GB RAM: the big one

Qwen 3.8 Flash-Next Coder through [Strata](https://github.com/Niko1221/Strata) (free). It keeps part of the model on the card and borrows your RAM for the rest.

**TESTED**: 41 to 46 tok/s writing code, 8 to 36 on plain text. It passed 5 of my 6 jobs and tied the sixth, beating the 27B. The catch: it took my free RAM from 19 GB down to 49 MB, so close everything else first. It's an 87 GB download.

## Tell me what you've got

Comment your card and what speed you get. I test the next model on whatever most of you are running.
