data definition language

실습은 ORACLE로 함

주요 데이터 유형
s는 길이
CHARACTERE(s) : 고정 길이 문자열. Oracle 최대 2000바이트. SQL 8000바이트 
ㄴ 실제 할당 변수값이 더 작다면 그 차이가 채워짐
VARCHAR(s) : 가변 길이 문자열. Oracle 최대 4000바이트. SQL 8000바이트
NUMERIC : 정수 실수 구별 x. 
ㄴ  ORACLE 은 NUMBER. SQL은 10가지의 숫자 타입이 있다
   ㄴ 전체 자리 수를 지정하고 소수 자리 지정. NUMBER(6.2) -> xxxxxx.xx
DATETIME : DATE

문자열 유형 별 비교(CHAR은 나머지 뒷부분을 공백으로 채우기 때문에)
CHAR                   VARCHAR
'AA' = 'AA '          'AA' != 'AA '

테이블 생성 규칙
:단수형 권고
:테이블 명 중복 x
:테이블 이름 지정, 각 칼럼 괄호로 묶기
:각 칼럼 ,로 구분 끝은 ;
:칼럼 뒤 데이터 유형 표기
