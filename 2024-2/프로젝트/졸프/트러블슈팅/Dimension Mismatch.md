### 제미나이 
#### 1. 원인 분석

- **Student (`MobileCLIP2-S4`)**: 출력 임베딩 차원이 **768**입니다.
    
- **Teacher 1 (`SigLIP2`)**: 출력 임베딩 차원이 **1152**입니다.
    
- **에러 메시지**: "768이랑 1152를 어떻게 `MSE Loss`(차이 계산)를 하라는 거야?"라며 화를 내는 중입니다.
    

지식 증류(Distillation)를 할 때는 스승과 제자의 체급(차원)이 다르기 때문에, 이를 맞춰주는 **선형 투영 레이어(Linear Projection Layer)**가 반드시 필요합니다.

---

#### 2. 해결 방법: Projection Layer 추가

`student_feats`(768)를 `teacher_feats`(1152) 크기로 뻥튀기해 주는 간단한 `Linear` 레이어를 추가해야 합니다. 스승이 둘이므로 각각에 맞는 레이어가 필요할 수 있습니다.

#### [Step 1] `main` 함수 내부에 레이어 정의

스승 모델들을 로드한 직후에 아래 코드를 추가하세요.

Python

```
# Teacher 1 (SigLIP2: 1152)용 프로젝션
proj_t1 = torch.nn.Linear(768, 1152).to(device) 
# 만약 Teacher 2 (ViT-L-14)의 차원이 또 다르다면? (보통 768)
proj_t2 = torch.nn.Linear(768, 768).to(device) # 차원이 같으면 사실 필요 없지만 구조상 필요할 수 있음

# 옵티마이저에 이 레이어들도 학습되도록 추가해야 합니다!
params = list(student_model.parameters()) + list(proj_t1.parameters()) + list(proj_t2.parameters())
optimiser = torch.optim.AdamW(params, lr=config.lr, weight_decay=config.weight_decay)
```

#### [Step 2] 학습 루프 내에서 차원 맞추기

학습 루프에서 `distillation_loss`를 호출하기 전에 차원을 변환해 줍니다.

Python

```
if teacher_models:
    for i, (weight, teacher) in enumerate(zip(config.distill_weights, teacher_models)):
        with torch.no_grad():
            t_img, t_txt = teacher(images, texts)
        
        # i번째 스승의 차원에 맞게 제자의 임베딩을 변환
        if i == 0: # SigLIP2 (1152)
            s_img_proj = proj_t1(img_embeds)
            s_txt_proj = proj_t1(txt_embeds)
        else: # ViT-L-14 (768)
            s_img_proj = proj_t2(img_embeds)
            s_txt_proj = proj_t2(txt_embeds)
            
        loss += weight * (distillation_loss(s_img_proj, t_img) + distillation_loss(s_txt_proj, t_txt))
```

---

#### 3. 더 쉬운 방법: Normalize & MLP (학부 수준 추천)

만약 레이어를 추가하는 게 복잡하다면, `config` 설정에서 `embed_dim`을 스승과 똑같이 맞춰보려 할 수 있지만, `MobileCLIP2-S4`의 기본 구조가 768이라 고정되어 있을 겁니다. 위와 같이 **Linear Layer**를 하나 끼워넣는 게 가장 정석적인 지식 증류 방식입니다.