# Lab 21 - Evaluation Report

**Học viên**: Nguyễn Quang Minh - 2A202600195  
**Ngày nộp**: 2026-05-07  
**Submission option**: B - GitHub + HuggingFace Hub

---

# 1. Setup

- **Base model**: `unsloth/Qwen2.5-3B-bnb-4bit`
- **Fine-tuning method**: QLoRA với Unsloth + PEFT
- **Dataset**: `5CD-AI/Vietnamese-alpaca-gpt4-gg-translated`
- **Dataset size**: 200 samples
  - Train: 180
  - Eval: 20
- **Formatting**: Alpaca instruction format
- **max_seq_length**: 1024
  - p50 = 227
  - p95 = 562
  - p99 = 704
- **GPU**: Tesla T4 - 15.6 GB VRAM
- **CUDA**: 12.8
- **PyTorch**: 2.10.0+cu128
- **Training framework**:
  - Unsloth 2026.5.2
  - TRL 0.15.2
  - Transformers 5.5.0
- **Training configuration**:
  - Batch size = 1
  - Gradient accumulation = 8
  - Effective batch size = 8
  - Epochs = 3
  - Learning rate = 2e-4
  - Scheduler = cosine
  - Optimizer = adamw_8bit
  - Quantization = 4-bit NF4
- **Total training time**: ~11.5 minutes
- **Estimated training cost**: ~$0.07 (@ $0.35/hr T4)
- **HF Hub link**:
  - https://huggingface.co/minh-nguyen-mn/qwen2.5-3b-vi-lab21-r16

---

# 2. Rank Experiment Results

| Rank | Alpha | Trainable Params | Train Time | Peak VRAM | Eval Loss | Perplexity |
|------|------|------------------|------------|-----------|-----------|------------|
| 8 | 16 | 1,843,200 | 3.60 min | 7.22 GB | 1.5577 | 4.75 |
| 16 | 32 | 3,686,400 | 4.27 min | 6.62 GB | 1.5161 | 4.55 |
| 64 | 128 | 14,745,600 | 3.58 min | 8.00 GB | 1.4768 | 4.38 |

## Observations

- Increasing LoRA rank improved evaluation loss and perplexity consistently.
- Rank 64 achieved the best perplexity (4.38), indicating the strongest adaptation capacity.
- Rank 8 already performed reasonably well despite training only ~0.06% of parameters.
- VRAM usage increased with rank size, especially for r=64.
- Training time differences between ranks were relatively small due to the lightweight dataset size.

---

# 3. Loss Curve Analysis

## Training Behavior

The training loss decreased steadily across all ranks:

- r=8: approximately 1.62 → 1.44
- r=16: approximately 1.61 → 1.39
- r=64: approximately 1.60 → 1.27

This indicates stable optimization without divergence.

## Overfitting Analysis

No major signs of overfitting were observed because:

- Training loss decreased smoothly.
- Evaluation perplexity also improved as rank increased.
- The dataset size was relatively small but the number of epochs (3) remained moderate.
- QLoRA regularization and low trainable parameter count reduced overfitting risk.

However, since evaluation during training was disabled on T4 to save VRAM, only final evaluation metrics were available.

---

# 4. Qualitative Comparison

## Example 1

### Prompt
> Giải thích khái niệm machine learning cho người mới bắt đầu.

### Base Model
Machine learning là một phân khúc của trí tuệ nhân tạo, nó tập trung vào việc thiết lập các mô hình máy móc để học tập từ dữ liệu và từ đó có thể dự đoán hoặc hành động.

### Fine-tuned Model
Machine learning là một bộ môn công nghệ máy tính dựa trên việc học tập và cải thiện các dự đoán dựa trên dữ liệu mà không có sự hướng dẫn trực tiếp từ người dùng.

### Nhận xét
Fine-tuned model tạo câu trả lời tự nhiên và rõ ràng hơn bằng tiếng Việt, với cấu trúc giải thích tốt hơn.

---

## Example 2

### Prompt
> Viết đoạn code Python tính số Fibonacci thứ n.

### Base Model
Sử dụng cách đệ quy cơ bản, thiếu xử lý input rõ ràng.

### Fine-tuned Model
Cung cấp implementation iterative đầy đủ hơn với:
- xử lý lỗi input
- logic rõ ràng
- Python style tốt hơn

### Nhận xét
Fine-tuned model tạo code thực tế và production-friendly hơn.

---

## Example 3

### Prompt
> Liệt kê 5 nguyên tắc thiết kế UI/UX.

### Base Model
Tập trung nhiều vào mô tả dài dòng.

### Fine-tuned Model
Danh sách concise hơn:
- chuyển đổi
- thích ứng
- đơn giản
- tương thích
- trực quan

### Nhận xét
Fine-tuned model trả lời ngắn gọn và có cấu trúc tốt hơn.

---

## Example 4

### Prompt
> Tóm tắt sự khác biệt giữa LoRA và QLoRA.

### Base Model
Giải thích tương đối đúng nhưng khá mơ hồ.

### Fine-tuned Model
Câu trả lời có cấu trúc kỹ thuật rõ hơn và nhắc đến optimization/regularization.

### Nhận xét
Fine-tuned model thể hiện domain adaptation tốt hơn cho chủ đề LLM fine-tuning.

---

## Example 5

### Prompt
> Phân biệt prompt engineering, RAG, và fine-tuning.

### Base Model
Giải thích chung chung.

### Fine-tuned Model
Phân biệt rõ hơn:
- Prompt engineering = thiết kế instruction
- RAG = retrieval external knowledge
- Fine-tuning = cập nhật model weights

### Nhận xét
Fine-tuned model có khả năng tổ chức kiến thức tốt hơn và trả lời theo hướng educational.

---

# 5. Conclusion về Rank Trade-off

Trong thí nghiệm này, rank 16 cho ROI tốt nhất giữa chất lượng và resource usage. So với rank 8, rank 16 cải thiện perplexity đáng kể nhưng chỉ tăng số lượng trainable parameters từ khoảng 1.8M lên 3.7M. Đồng thời, VRAM usage vẫn nằm trong giới hạn an toàn của Tesla T4.

Rank 64 đạt kết quả tốt nhất về perplexity (4.38), cho thấy khả năng học biểu diễn mạnh hơn. Tuy nhiên, improvement từ rank 16 → rank 64 nhỏ hơn improvement từ rank 8 → rank 16. Điều này thể hiện hiện tượng diminishing returns: tăng rank tiếp tục cải thiện chất lượng nhưng mức cải thiện bắt đầu giảm dần.

Ngoài ra, rank 64 sử dụng gần 15M trainable parameters và VRAM cao hơn đáng kể. Với dataset nhỏ chỉ gồm 200 samples, rank quá lớn có thể không tận dụng hết năng lực biểu diễn và có nguy cơ overfitting nếu train lâu hơn.

Nếu deploy production trên hạ tầng giới hạn GPU như T4 hoặc inference edge systems, tôi sẽ chọn rank 16 vì:
- perplexity tốt
- VRAM thấp
- training ổn định
- inference efficient
- checkpoint nhỏ gọn

Nếu có dataset lớn hơn và GPU mạnh hơn, rank 64 có thể trở thành lựa chọn tốt hơn cho chất lượng tối đa.

---

# 6. What I Learned

- QLoRA cho phép fine-tune LLM 3B trên GPU T4 16GB rất hiệu quả bằng quantization 4-bit.
- LoRA rank ảnh hưởng trực tiếp đến trade-off giữa model quality và compute/memory usage.
- Unsloth giúp tăng tốc đáng kể training và giảm VRAM consumption trên Colab free tier.
- Việc chọn max sequence length theo p95 token distribution giúp tối ưu memory tốt hơn.
- Fine-tuning cải thiện instruction-following và response quality ngay cả với dataset relatively nhỏ.
- Adapter-based training giúp workflow production-ready hơn nhiều so với full fine-tuning.

---

# 7. Links

## HuggingFace Hub
- https://huggingface.co/minh-nguyen-mn/qwen2.5-3b-vi-lab21-r16

## Google Colab Notebook
- https://github.com/minh-nguyen-mn/lab21_2A202600195/blob/main/notebook.ipynb

## Result Files
- https://github.com/minh-nguyen-mn/lab21_2A202600195/blob/main/results/qualitative_comparison.csv
- https://github.com/minh-nguyen-mn/lab21_2A202600195/blob/main/results/rank_experiment_summary.csv
