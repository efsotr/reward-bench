# info 目录说明

这里汇总运行 RewardBench 时产生的补充信息，方便查看结果和脚本逻辑。

## 结果存放在哪里？

评估脚本会通过 `rewardbench.save_to_hub` 将结果写到本地 `results/` 目录（例如 `results/eval-set/<model>.json`），同时可选择上传到 Hub。若只想本地查看，可在调用时设置 `local_only=True`（或使用 `--do_not_save` 仅跳过上传）。

## `run_generative_model.py`（对应 `scripts/run_generative.py`）如何处理 judge 顺序？

- 在 API 路径中，`get_judgement` 会先用 50% 概率交换 Answer A/B，以消除位置偏置，并记录哪一侧应当获胜（`winner_text` / `loser_text`）。不同 judge 模型的输出会在内部解析为 A/B 标签，例如默认提示词要求返回 `[[A]]` 或 `[[B]]`（见 `rewardbench/generative.py` 中的 `prompt_v2`）。解析后的标签再映射回原始顺序，最终写入 `results` 列为 1（原始优选）、0（原始劣选）或 0.5（无效/平局）。
- 在本地 vLLM 路径中，`format_judgements` 同样会随机交换回答并记录 `is_shuffled`。随后 `process_shuffled` 使用该标记将模型输出映射回初始顺序，生成同样含偏置纠正的 `results`。

因此，无论运行路径如何，保存下来的得分都已经过随机打乱和映射处理，不需要额外调整顺序。
