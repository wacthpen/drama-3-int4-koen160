# Drama-3 int4 KoEn160 (Korean/English expert-pruned, based on Fish Audio Drama 3)

**Weights (17.7 GB) · 무게 파일: https://huggingface.co/watchpen/drama-3-int4-koen160**

This repository holds the model card, license, notice and the SGLang-Omni Hadamard 8-bit patch. GitHub cannot hold the weights; download them from Hugging Face.
이 저장소에는 설명·라이선스·NOTICE·하다마드 8비트 패치가 있습니다. 무게는 깃허브 용량 한도 때문에 허깅페이스에서 받으세요.

**Built with Fish Audio.** An unofficial Korean/English variant of
[fishaudio/drama-3-int4](https://huggingface.co/fishaudio/drama-3-int4), made to run on a single
24 GB GPU (RTX 3090). Not endorsed by Fish Audio.

- **Runs within 24 GB of VRAM.**
- **3–4× faster than the cloud server.**

## What changed

| | fishaudio/drama-3-int4 | this model |
|---|---|---|
| Routed experts per MoE layer | 256 | **160** (Korean/English) |
| Weights on disk | 23 GB | 17 GB |
| lm_head · embed_tokens · audio decoder linears | bf16 | bf16 on disk, **Hadamard int8 at load** with the patch below |

- **Expert pruning.** We ran the original model on about 150 Korean and English lines, counted which
  routed experts the router picked in each layer, and kept the 160 most used. No retraining.
  Languages other than Korean and English were not kept in mind and are likely worse.
- **Hadamard 8-bit (optional, runtime).** `sglang-omni-patch/drama3-hadamard8.patch` makes
  SGLang-Omni rotate each 256-wide group with a Hadamard matrix and store the remaining bf16
  layers as int8 (W8A8) while loading, saving about 1.9 GB of GPU memory (relative error 1.2–1.5 %).

## Measured (RTX 3090 24 GB, Hadamard 8-bit on)

- Server ready in 54 s; one line synthesised at **3.1–3.3× real time** (original int4 on the same
  card, with the codec on CPU to fit: about 0.9×).
- Peak GPU memory 23.4 GB.
- Settings we used: engine memory fraction 0.58, vocoder 0.18, reference reading 4096 MiB,
  at most 560 frames (≈26 s) per line, vocoder CUDA graph off, reference encoder and vocoder in fp32 on GPU.

## Run with SGLang-Omni

```bash
git clone https://github.com/sgl-project/sglang-omni && cd sglang-omni
git checkout 1240a6993c8073593aa0ad074859ccee2797adff
git apply /path/to/this/repo/sglang-omni-patch/drama3-hadamard8.patch   # optional: Hadamard int8
pip install -e .
DRAMA3_HADAMARD8=1  # turn the int8 path on (unset or 0 = original bf16 path)
# then serve this folder the same way as fishaudio/drama-3-int4
```

Without the patch the model loads as plain int4 + bf16 (about 1.9 GB more GPU memory).

## Known limits

- Like the original, it sometimes keeps generating past the end of the text; cap the frames per line.
- Only Korean and English were checked by ear.

## License

Fish Audio Community License 1.0 — see `LICENSE` and `NOTICE`. In short: free for research and
non-commercial use, and for commercial use below the revenue threshold in the license; offering it
as a hosted service needs a separate agreement with Fish Audio. You may not use it, or its outputs,
to imitate an identifiable person's voice without consent, or to train or distill other speech or
generative models.

## 한국어

피시 오디오 드라마3 int4 판에서 한국어·영어에 쓰이는 전문가만 남긴(256 → 160) 판입니다. 하다마드 회전 8비트
패치를 쓰면 RTX 3090 한 장에서 실시간 3배 빠르기로 돕니다. 피시 오디오 공식 판이 아닙니다.

- **VRAM 24GB 안에서 구동됩니다.**
- **클라우드 서버보다 3~4배 빠릅니다.**
