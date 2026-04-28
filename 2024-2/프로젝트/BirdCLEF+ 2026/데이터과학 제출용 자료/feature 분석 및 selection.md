#### train.csv
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

### train.csv preprocess list
1. **primary label, secondary labels**의 이름이 학명, 일반 이름, inat_tax_id가 모두 혼합
	- primary label 값을 inat_taxon_id로 통일해야한다.
	- train.csv의 inat column에 없을 경우 taxonomy.csv 와 대조해서 찾아봐야한다.
2. rating에 60.1236, url 등 이상한 값이 들어가있어서 이건 쳐내야 한다. 
	- 0으로 초기화하는 것이 좋을 것 같다.

### train_soundscapes_labels
1. primary label : ';'를 구분자로 종이 나열되어 있다. 
	- 마찬가지로 



