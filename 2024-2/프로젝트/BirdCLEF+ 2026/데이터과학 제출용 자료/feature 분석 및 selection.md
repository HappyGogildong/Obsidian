### train.csv
	primary_label  : 녹음 파일의 지배적인 조류 ID (정수 숫자)
	secondary_labels  : 배경에서 작게 들리는 다른 새 종 ID. 실제 확인 결과 거의 빈 값
	type : 새 소리의 종류. call, son, flight call 등
		 여러 소리가 관측될 경우 리스트 형식
	latitude : 위도
	longitude : 경도
	scientific_name : 학명
	common_name : 일반적 명칭
	class_name : 종 분류 이름
	inat_taxon_id : iNat 데이터베이스 고유 식별번호
	author : 수집한 사람 이름
	license : 음원 저작권 라이선스
	rating : 녹음 품질에 대한 별점.
	url : 원본 소리 파일 링크
	filename : 링크에서 실제 오디오 파일 경로와 이름
	collection: 해당 녹음이 포함된 데이터 컬렉션 이름

### train.csv 
1. **primary label, secondary labels**의 이름이 학명, 일반 이름, inat_tax_id가 모두 혼합
	- sample submission의 column도 모두 혼합되어 사용된다.
	- 
2. rating에 60.1236, url 등 이상한 값이 들어가있어서 이건 쳐내야 한다. 
	- 0으로 초기화하는 것이 좋을 것 같다.

### train audio
1. 각 클래스별로 가진 오디오의 갯수, 품질이 모두 다르다(32000khz는 동일)
2. train.csv의 메타데이터를 참고 가능 
### train_soundscapes_labels.csv
1. primary label : ';'를 구분자로 종이 나열되어 있다. 
	- 마찬가지로 id로 통일되어 있지 않기 때문에 변경하는 작업이 필요하다
		- class indexing 하면 0-233 숫자로 통일가능
	- train soundscapes에 없는 종이 test에 등장하거나, 그 반대도 가능.`

### sample_submission.csv
1. 각 종이 제시되어 있고, 이 순서에 맞게 제출 csv도 순서를 맞추어야한다.

### taxonomy.csv
1. 235종을 나열해놓았다. 
2. 이 csv의 row 순서가 sample_submission의 column 순서와 동일하므로
	- 클래스별 threshold, 신경망을 위해 종 tax id, sono, 학명을 숫자로 변경할 때 사용하면 유용할 것 같다
	- 그리고 출력 서식도 이것으로 맞추자
3. 곤충의 경우엔 다른 클래스(primary label 기준)이더라도 같은 tax_id를 사용하는 경우가 있다. 따라서 모든 라벨을 tax id로 변환 후 다시 숫자로 변환하지 말고 그대로 primary label 값을 숫자로 별명을 짓는 식으로 가야한다

### 총 요약
1. primary label을 그대로 인덱싱해서 연산
	- 인덱싱 테이블 따로 적어놓자(내가 보기 편하게)
	- train.csv
2. class(234종을 말하는 것)별 train 샘플 데이터 양, 질 분류
	 - 해당 결과를 토대로 augmentation hyper parameter 책정
3. train_soundscape를