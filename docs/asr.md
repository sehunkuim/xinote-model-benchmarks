# Streaming ASR: Qwen3-ASR

## 1. 현재 v27 계열: 추가 학습 전후

비교는 **v26m2 merged4800 → v27 merged8000**입니다. 같은 1초 스트리밍 설정과 각 100발화, 같은 참조 단어 수를 확인했습니다. 몽골어 4개 공통 부분집합을 모두 표시합니다.

| 평가 / 비교 | 지표 | n / 유효 n | 초기·기준 | 학습·변경 후 | 변화 |
|---|---|---:|---:|---:|---:|
| MN-online-speech · v26m2-to-v27 | WER | 100 / 100 | 52.88% | 40.40% | -12.48 pp |
| MN-broadcast · v26m2-to-v27 | WER | 100 / 100 | 17.75% | 16.84% | -0.91 pp |
| MN-subtitled-speech · v26m2-to-v27 | WER | 100 / 100 | 52.12% | 27.97% | -24.15 pp |
| MN-read-speech · v26m2-to-v27 | WER | 100 / 100 | 23.76% | 28.62% | +4.86 pp |

MN-read-speech에서는 WER가 **23.76% → 28.62%**로 악화했습니다. 가장 좋아진 한 부분집합을 전체 언어 성능으로 일반화하지 않습니다.

| v27 학습 설정 | 값 |
|---|---|
| Starting checkpoint | v26m2 merged4800; already fine-tuned 1.7B model |
| Loss implementation | weighted next-token cross-entropy on eligible target tokens; ambiguous tokens masked; prefix weight 0.3 |
| Optimizer | AdamW; implementation betas 0.9/0.98; weight decay 0.01; grad clip 1.0 |
| Learning rates | LoRA 2e-5; audio 1.2e-5; new embedding rows 1e-4 |
| LoRA | rank 64; alpha 128; dropout 0.05 |
| Schedule | warmup 200 steps; cosine; min LR ratio 0.1 |
| Budget | 8,000 steps; gradient accumulation 1 |
| Dynamic batch | 720 audio seconds, at most 144 examples / batch |
| Precision | base BF16; trainable parameters and AdamW state FP32 in inspected implementation |
| Training mixture modes | 1-second chunks; offline probability 0.35; automatic language probability 1.0 |
| Augmentation | probability 0.1; reverb/noise branch probability 0.45 each; SNR -3 to 20 dB |
| Seed | 20261006 |
| Evaluation | 1-second chunks; unfixed chunks/tokens 2/5; warmup speech 6 seconds; gen_batch 16; auto language enabled |

실행 시 설정된 `qwen17_v27` 가중치와 학습 export의 SHA-256이 동일했습니다:
`1152b401d7ff4022f0ca5a9279cb5d4d5b79512a3c635a0bcb59a1293857ec49`.

배포 폴더의 오래된 run-args 표시는 v19를 가리켜서 그대로 인용하지 않았습니다. 학습 run v27의 별도 args와 실제 weights 지문으로 확인했습니다. Optimizer/loss 구현은 현재 보존 소스에서 확인했으며, 해당 run 당시 소스의 완전한 고정 snapshot은 확보하지 못했습니다.

## 2. 원본 0.6B → LoRA: 과거 실험

| 평가 / 비교 | 지표 | n / 유효 n | 초기·기준 | 학습·변경 후 | 변화 |
|---|---|---:|---:|---:|---:|
| KO-lecture · original-to-LoRA6000 | script_cer | 914 / 914 | 9.44% | 7.69% | -1.75 pp |
| KO-lecture · original-to-LoRA6000 | mixed_cer | 914 / 914 | 9.77% | 8.13% | -1.63 pp |
| KO-EN-code-switch · original-to-LoRA6000 | script_cer | 600 / 600 | 28.20% | 21.37% | -6.84 pp |
| KO-EN-code-switch · original-to-LoRA6000 | mixed_cer | 600 / 600 | 33.55% | 25.72% | -7.83 pp |
| KO-heldout · original-to-LoRA6000 | script_cer | 600 / 600 | 18.84% | 14.90% | -3.95 pp |
| KO-heldout · original-to-LoRA6000 | mixed_cer | 600 / 600 | 22.87% | 24.56% | +1.69 pp |
| KO-read-speech · original-to-LoRA6000 | script_cer | 457 / 457 | 5.31% | 3.51% | -1.81 pp |
| KO-read-speech · original-to-LoRA6000 | mixed_cer | 457 / 457 | 5.41% | 3.51% | -1.90 pp |
| KO-conversation · original-to-LoRA6000 | script_cer | 100 / 100 | 14.08% | 10.41% | -3.67 pp |
| KO-conversation · original-to-LoRA6000 | mixed_cer | 100 / 100 | 14.22% | 10.45% | -3.77 pp |
| EN-read-speech · original-to-LoRA6000 | wer | 240 / 240 | 2.24% | 1.90% | -0.34 pp |

이 표는 v27 1.7B와 별개의 과거 0.6B 실험입니다. lora_b는 이전 lora_a adapter를 초기화하여 이어 학습했습니다. LR 6e-5, audio LR 3e-5, rank/alpha/dropout 64/128/0.05, warmup 200, 6,000 steps, dynamic audio budget 480초 / 최대 96개, seed 20260929입니다.

같은 1초 chunk 비교의 아카이브이며, 생략된 옵션의 기본값과 당시 실행 소스 전체가 고정돼 있지는 않습니다. KO-heldout의 mixed CER 등 하락한 지표도 제외하지 않았습니다. 과거 v14→v15, offline→streaming, serving batching 최적화를 이 표의 동일 학습 효과로 섞지 않습니다.

근거: `ASR-v26-eval`, `ASR-v27-eval`, `ASR-v27-config`, `ASR-06-config`, `asr06_before`, `asr06_after`.
