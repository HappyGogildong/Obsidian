### 1. RLVR의 부상

2025 이전의 트렌드
pretraining -> SFT -> RL

2025이후, RLVR(Reinforcement Learning from Verifiable Rewards)이 부상함.

RLVR은 verifiable(검증 가능한) 문제들에 RL을 적용하는 것이다.수학 문제, 퍼즐과 같은

특히 RLVR은 capability/$가 뛰어나서, 기존에 pretraining에 할당되던 컴퓨팅 자원까지 여기에 투자하기 시작했음.
그리고 이 RLVR에서 테스트 타임 연산량을 늘려 추론 과정과 생각하는 시간을 증가시킴으로써 모델의 능력을 조절할 수 있는 새로운 knob와 대응되는 스케일링 법칙을 얻게 됨.

### 2. Ghost vs Animals/Jagged Intelligence
2025년은 대부분의 사람이 LLM 지능의 형태에 직관적인 감을 얻기 시작한 시기다.
LLM 스택은 진화보다는 summoning ghost에 가깝다. LLM 스택은 개발주체별로 다른 문법, 데이터, 최적화를 사용한다. 그래서 동물의 진화와 같이 생각하는 것은 부적절하다. 그들(LLM)들은 같은 entity로 보기엔 어렵다.

RLVR은 학습된 도메인 부근의 성능은 좋지만 다른 곳은 그렇지 않은 jaged performance를 보여준다. Polymath에 천재적이면서도 초등학생과 같은 혼란스러운 인지 능력을 가져 jailbreak를 통해 데이터를 유출할 가능성이 있다. 

2025년 진행된 벤치마크 결과를 보고 이런 냉담한 반응을보이게 되었는데,
벤치마크 결과가 좋지 않았던 핵심 이유는 대부분 벤치마크는 검증 가능한 환경이기에
RLVR과 혹은 더 약한 합성 데이터 생성에 취약하다는 점이다.(game이 가능하다)

### 3. LLM app의 새로운 layer

Cursor와 같은 LLM app은 특정 분야의 LLM call을 묶고 관리한다(지휘한다)

1. Context Engineering을 한다
2. 성능과 비용 사이의 균형을 맞추며 복잡한 DAG로 연결된 LLM사이의 호출을 조율한다
3. Human in the loop을 위해 앱 별 GUI를 제공한다
4. "autonomy slider"를 제공한다


### 4. Claude Code / AI that lives on your computer

Claude Code는 LLM Agent의 모습을 설득력 있게 보여준 첫 번째 사례다.
CC(Claude Code)는 로컬 환경의 데이터에 접근가능한 agent다.
클라우드 환경을 누비는 agent가 AGI endgame처럼 느껴지지만, 
우리는 그 중간 단계에 살고 있다. 그래서 agent를 각 개발자의 셋업에 맞게 설정하는 것이 중요했고, CC는 그 우선순위를 정확히 캐치해냈다. CC는 컴퓨터에 살고 있는 ghost다.


