rouzeta
===

- 설명
  - FST에 기반한 형태소 분석기 [Rouzeta](https://shleekr.github.io/)의 사용법

- Rouzeta 다운로드
  - [rouzeta](https://shleekr.github.io/public/data/rouzeta.tar.gz)
  ```
  $ tar -zxvf rouzeta.tar.gz
  $ ls -R
  .:
  Rouzeta  Tagger

  ./Rouzeta:
  korean.lexc  kormoran.script  morphrules.foma  splithangul.foma

  ./Tagger:
  koreanuni.xml  korfinaluni.fst  korinvert.sym  testme.txt  worduniprob.sym
  ```

- Rouzeta 설치시 필요한 패키지
  - [foma](https://code.google.com/archive/p/foma/)
  ```
  * stack full을 피하기 위해서, int_stack.c를 수정하고 다시 컴파일해야함.
  $ cd foma-0.9.18
  $ vi int_stack.c
  ...
  #define MAX_STACK 2097152*10
  #define MAX_PTR_STACK 2097152*10

  static int a[MAX_STACK];
  static int top = -1;
  ...
  $ make ; sudo make install
  ```
  - [xerces-c](http://xerces.apache.org/xerces-c/download.cgi)
  ```
  $ cd xerces-c-3.1.4
  $ ./configure ; make ; sudo make install
  ```
  - [OpenFST](http://www.openfst.org/twiki/bin/view/FST/WebHome)
  ```
  * <주의> 최신 버전으로 하면, kyfd 컴파일시 오류가 발생한다.
  $ openfst-1.3.2
  $ ./configure ; make ; sudo make install
  ```
  - [kyfd](http://www.phontron.com/kyfd/)
  ```
  $ cd kyfd-0.0.5
  $ ./configure ; make ; sudo make install
  ```

- 형태소분석
```
$ cd KFST/Rouzeta
$ forma
foma[0]: source kormoran.script
....
defined AlternationRules: 26.7 MB. 360257 states, 1750440 arcs, Cyclic.
defined SingleWordPhrase: 42.9 MB. 127415 states, 2797946 arcs, Cyclic.
defined Delim: 202 bytes. 2 states, 1 arc, 1 path.
defined NormalizeDelim: 364 bytes. 2 states, 4 arcs, Cyclic.
defined WPWithMark: 42.9 MB. 127417 states, 2798456 arcs, Cyclic.
defined SentenceWithMark: 42.9 MB. 127417 states, 2798457 arcs, Cyclic.
defined DeleteMarkUp: 376 bytes. 1 state, 3 arcs, Cyclic.
defined DeleteMarkDown: 376 bytes. 1 state, 3 arcs, Cyclic.
defined Sentence: 43.0 MB. 127416 states, 2806708 arcs, Cyclic.
43.0 MB. 127416 states, 2806708 arcs, Cyclic.
foma[1]: up
apply up> 형태소분석은
형태소/nc분석/nc은/nc
형태소/nc분석/nc은/pt
형태소/nc분/nc석/nc은/nc
형태소/nc분/nc석/nc은/pt
....
```

- 형태소분석용 FST를 저장
```
$ cd KFST/Rouzeta
$ vi kormoran.script
# 아래와 같이 주석을 제거한다.
save stack kor.stack				! 만들어진 FST를 binary로 저장하는 방법
write att > "kor.fomaatt"			! att 형식으로 파일을 저장하는 방법
invert net							! 표층형 -> 어휘형으로 바뀌기 위한 방법
write att > "korinvert.fomaatt"	! 바뀌어진 FST를 att형식으로 저장하는 방법
$ foma -f kormoran.script
```

- 태깅
```
$ cd KFST/Tagger
# 직접 설치한 kyfd를 사용한다.
$ rm kyfd
$ cat testme.txt | ./kyfd koreanuni.xml
--------------------------
-- Started Kyfd Decoder --
--------------------------

Loaded configuration, initializing decoder...
Loading fst korfinaluni.fst...
 Done initializing, took 0 seconds
Decoding...
나 /np 는 /pt <space> 학 교 /nc 에 서 /pa <space> 공 부 /na 하 /xv _ㅂ 니 다 /ef . /sf
.선 /nc 을 /po <space> 긋 /irrs /vb 어 /ex <space> 버 리 /vx 었 /ep 다 /ef . /sf
.고 맙 /irrb /vj 었 /ep 다 /ef . /sf
.나 /np 는 /pt <space> 답 /nc 을 /po <space> 모 르 /irrl /vb 아 /ec . /sf
.지 나 /vb _ㄴ /ed <space> 1 8 /nb 일 /nc <space> 하 오 /nc <space> 3 /nb 시 /nc <space> 경 남 /nr <space> 마 산 시 /nr
.색 /nc 이 /ps <space> 하 얗 /irrh /vj 어 서 /ef <space> 예 쁘 /vj 었 /ep 다 /ef . /sf
.일 찍 /ad <space> 일 어 나 /vb 는 /ed <space> 새 /nc 가 /ps <space> 피 곤 /ns 하 /xj 다 /ef ( /sl 웃 음 /nc ) /sr . /sf
.꽃 /nc 이 /ps <space> 핀 /nc <space> 곳 /nc 을 /po <space> 알 /vb 고 /ec 있 /vj 다 /ef . /sf
.이 것 /nm 은 /pt <space> 사 과 /nc 이 /pp 다 /ef . /sf
.상 자 /nc 를 /po <space> 연 /nc <space> 사 람 /nc 은 /pt <space> 그 /np 이 /pp 다 /ef . /sf
.사 과 /nc _ㄹ /po <space> 먹 /vb 겠 /ep 다 /ef . /sf
.향 약 /nc 은 /pt <space> 향 촌 /nc 의 /pd <space> 교 육 /nc 과 /pc <space> 경 제 /nc 를 /po <space> 관 장 /nc 해 /nc <space> 서 원 /nc 을 /po <space> 운 영 /na 하 /xv 면 서 /ef <space> 중 앙 /nc <space> 정 부 /nc <space> 등 용 문 /nc 인 /nc <space> 대 과 /nc <space> 응 시 자 격 /nc 을 /po <space> 부 여 /na 하 /xv 는 /ed <space> 향 시 /nc 를 /po <space> 주 관 /nc 하 고 /pq <space> 흉 년 /nc 이 /ps <space> 들 /vb 면 /ex <space> 곡 식 /nc 을 /po <space> 나 누 /vb 는 /ed <space> 상 호 부 조 /nc 와 /pc <space> 작 황 /nc 에 /pa <space> 따 르 /vb _ㄴ /ed <space> 소 작 료 /nc <space> 연 동 적 용 /nc 을 /po <space> 정 하 /vb 는 가 /ef <space> 하 /vb 면 /ex <space> 풍 속 사 범 /nc 에 /pa <space> 대 하 /vb 어 /ex <space> 형 벌 /nc 을 /po <space> 가 하 /vb 는 /ed <space> 사 법 부 /nc <space> 역 할 /nc 까 지 /px <space> 담 당 /na 하 /xv 었 었 /ep 다 /ef . /sf
. Done decoding, took 0 seconds
```
