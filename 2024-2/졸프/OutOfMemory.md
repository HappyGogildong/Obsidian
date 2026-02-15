torch.OutOfMemoryError: CUDA out of memory. Tried to allocate 512.00 MiB. GPU 0 has a total capacity of 11.99 GiB of which 0 bytes is free. Of the allocated memory 23.95 GiB is allocated by PyTorch, and 193.86 MiB is reserved by PyTorch but unallocated. If reserved but unallocated memory is large try setting PYTORCH_ALLOC_CONF=expandable_segments:True to avoid fragmentation.  See documentation for Memory Management  (https://pytorch.org/docs/stable/notes/cuda.html#environment-variables)


### Gemini 의견
결국 올 것이 왔군요. **OOM(Out of Memory)** 에러입니다.

사용 중인 GPU 메모리는 **12GB(RTX 3060/4070 급)**인데, 현재 설정으로는 그 두 배인 **24GB** 정도를 요구하고 있습니다. 원인은 명확합니다. **거대한 스승 모델 2개(`SigLIP2`, `ViT-L-14`)를 동시에 띄운 상태에서 학습까지 하려니** 메모리가 버티지 못하는 것입니다.

학부생 프로젝트 수준에서 이 문제를 해결할 수 있는 **현실적인 최적화 방법**들을 정리해 드릴게요. 순서대로 적용해 보세요.

---

#### 1. 배치 사이즈(Batch Size) 대폭 축소

가장 확실하고 빠른 방법입니다. `config.yaml`에서 `batch_size`를 절반 이하로 줄이세요.

- **기존:** (예: 64 또는 32)
    
- **수정:** **`8` 또는 `4`**까지 줄여보세요.
    

> **팁:** 배치 사이즈가 너무 작아지면 학습이 불안정해질 수 있는데, 이때는 `gradient_accumulation_steps`를 활용해 실제 배치는 작게, 업데이트는 크게 가져가는 전략을 씁니다.

---

#### 2. 혼합 정밀도(Mixed Precision, AMP) 사용

이미 `config.yaml`에 `amp: bool` 설정이 있는 것 같은데, 이게 `True`로 되어 있는지 확인하세요. `FP16`을 사용하면 메모리 사용량을 거의 절반으로 줄일 수 있습니다.

- **config.yaml 수정:** `amp: True`
    

---

#### 3. 스승 모델 하나만 사용 (선택과 집중)

XR2 Gen 2용 모델을 만드는 게 목적이라면, 스승을 둘 다 쓰는 것보다 하나만 확실히 쓰는 것이 나을 수 있습니다.

- **추천:** `ViT-SO400M-14-SigLIP2` 하나만 남기고 `distill_weights`를 `[1.0]`으로 수정해 보세요.
    
- 메모리 여유가 생기면 그때 다시 두 번째 스승을 추가하는 식으로 단계별로 접근하는 것이 디버깅에 유리합니다.
    

---

#### 4. `PYTORCH_ALLOC_CONF` 설정

에러 메시지에서 추천한 대로 메모리 파편화를 방지하는 환경 변수를 설정하세요. 파이썬 코드 최상단(`import torch` 직후)에 넣으면 됩니다.

Python

```
import os
os.environ["PYTORCH_ALLOC_CONF"] = "expandable_segments:True"
```

---

#### 5. 불필요한 메모리 점유 해제

코드 내에서 모델 로딩 순서를 최적화하고, 필요 없는 텐서는 `del`로 지워야 합니다.

Python

```
# train.py 모델 로드 직후
import torch
torch.cuda.empty_cache() # 캐시 비우기
```

### 해결책
제미나이의 4,5 번을 적용해보고 먼저 2번 방법을 사용해보기로 했다
```
import os
os.environ["PYTORCH_ALLOC_CONF"] = "expandable_segments:True"
```

```
torch.cuda.empty_cache()
# Load student
...
# Load Teacher
...
torch.cuda.empty_cache()
```

fp32를 fp16으로 변경
```
kwargs = {
                'model_name': name,
                'pretrained': tag,
                'precision': 'fp16'
            }
```
