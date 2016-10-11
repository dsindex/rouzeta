### Rouzeta
- 설명
  - FST에 기반한 한국어 형태소 분석기 [Rouzeta](https://shleekr.github.io/)의 사용법

- Rouzeta 설치시 필요한 패키지
  - [foma](https://code.google.com/archive/p/foma/)
  ```
  # stack full을 피하기 위해서, int_stack.c를 수정하고 다시 컴파일해야함.
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
  - [openfst](http://www.openfst.org/twiki/bin/view/FST/WebHome)
  ```
  # <주의> 최신 버전으로 하면, kyfd 컴파일시 오류가 발생한다.
  $ openfst-1.3.2
  $ ./configure ; make ; sudo make install
  ```
  - [kyfd](http://www.phontron.com/kyfd/)
  ```
  $ cd kyfd-0.0.5
  $ ./configure ; make ; sudo make install
  ```

- Rouzeta 다운로드 & 소스 설명
  - [rouzeta](https://shleekr.github.io/public/data/rouzeta.tar.gz)
  ```
  $ tar -zxvf rouzeta.tar.gz
  $ ls -R
  .:
  Rouzeta  Tagger

  ./Rouzeta:
  korean.lexc  kormoran.script  morphrules.foma  splithangul.foma

  ./Tagger:
  koreanuni.xml  korfinaluni.fst  korinvert.sym  kydf testme.txt  worduniprob.sym
  ```
  - 소스 설명
  ```
  - Rouzeta
    - korean.lexc
	  형태소들과 각 형태소들의 접속 네트워크가 있는 사전
	  예를 들어, 

      LEXICON Root
        ncLexicon ; ! 보통명사
	    ...
        acLexicon ; ! 접속부사

      LEXICON acLexicon ! 접속부사
        고로/ac	acNext ;
        곧/ac	acNext ;
        그나/ac	acNext ;
		하지만/ac acNext ;
		...

      LEXICON acNext
        finLexicon ;
        srLexicon ;
        pxLexicon ;
        nnLexicon ;
		...

      위와 같이 기술하면, '고로/ac #', '곧/ac "/sr', '하지만/ac 은/px', '하지만/ac 첫째/nn'
	  등의 형태소 sequence가 valid하다는 의미이다.
	  일반적으로 어떤 태그와 어떤 태그가 결합가능한지 여부를 여기에 기술한다.
	  
	- morphrules.foma
	  어휘형과 표층형간의 규칙이 적혀 있는 룰 파일
	  예를 들어, 

      define IrrConjD		%_ㄷ -> %_ㄹ || _ %/irrd %/vb ㅇ ;

	  IrrConjD 규칙은 'ㄷ-불규칙'을 기술해서  아래와 같이 변환가능하게 한다.
	  'ㄷ ㅏ %_ㄷ  irrd vb ㅇ' -> 'ㄷ ㅏ %_ㄹ  irrd vb ㅇ'

	- splithangul.foma
	  utf-8 한글 음절을 자소로 분리하는 규칙
	  예를 들어,

	  '형 -> ㅎ ㅕ %_ㅇ'

      음절을 자소단위로 분리하는 규칙이다. '%_ㅇ'에서 '%_'는 종성을 의미한다. 

	- kormoran.script
	  위 3가지를 가지고 하나의 유한 상태 변환기 (finite state transducer)를 만드는 파일
	  즉, 형태소 분석용 FST(Rouzeta FST)를 만든다.
	  예를 들어, 

	  define SingleWordPhrase		Lexicon .o. SplitHangul .o.					    ! 사전으로부터 자소 분리
	  							    AlternationRules .o.						    ! 변환규칙 적용
								    RemoveTags .o.								    ! 품사/불규칙 태그 삭제
								    SplitHangul.i .o.							    ! 자소를 한글로 바꿈
								    ~$[Syllable] .o. ~$[KorChar] .o. ~$[CodaOnly] ; ! 자소열, 자모만, 종성 글자 제거

	  이 코드는 korean.lexc에 기술된 Lexicon을 자소단위로 분리하고
	  morphrules.foma의 변환규칙을 적용한 다음, 이 결과를 다시 음절단위로 변환하는
	  FST를 구성한다.

	  korean.lexc에 있는 엔트리를 가지고 설명하면 아래와 같다.

	  LEXICON vbNext
	    ...
		epLexicon
		...
	  LEXICON epLexicon
	    ...
		efLexicon

      깨닫/irrd/vb	vbNext ; ! vbLexicon, 동사
      았/ep	epNext ; ! epLexicon, 선어말어미
	  다/ef efNext ; ! efLexicon, 어말어미

	  vbLexicon             epLexicon    efLexicon
	  ---------------------------------------------
	  {깨 닫 /irrd /vb}  ->  {았 /ep}  ->  {다 /ef}

	  Lexicon 네트웍을 구성하면 vbLexicon의 모든 엔트리와 epLexicon의 모든 엔트리의 연결이 생성되어 있을 것이고
	  epLexicon과 efLexicon도 마찬가지다. 
	  여기서 음절을 자소로 분리한다음 IrrConjD 규칙을 적용하면

	  ㄲ ㅐ ㄷ ㅏ %_ㄷ /irrd /vb ㅇ ㅏ %_ㅆ /ep ㄷ ㅏ /ef
	  => 
	  ㄲ ㅐ ㄷ ㅏ %_ㄹ /irrd /vb ㅇ ㅏ %_ㅆ /ep ㄷ ㅏ /ef

	  이 결과에서 '품사/불규칙 태그'등을 삭제해야 다시 자소를 음절로 바꿀 수 있으므로, 삭제하면

	  ㄲ ㅐ ㄷ ㅏ %_ㄹ /irrd /vb ㅇ ㅏ %_ㅆ /ep ㄷ ㅏ /ef
	  =>
	  ㄲ ㅐ ㄷ ㅏ %_ㄹ ㅇ ㅏ %_ㅆ ㄷ ㅏ

	  이것을 다시 음절단위로 변환하자.

	  ㄲ ㅐ ㄷ ㅏ %_ㄹ ㅇ ㅏ %_ㅆ ㄷ ㅏ
	  =>
	  깨 달 았 다

	  composition 결과물에서 불필요한 노이즈(자소열, 자모만, 종성 글자 등등)는 제거한다.


  - Tagger
    - korfinaluni.fst
	  형태소 분석용 FST를 inverse한 FST와 와 unigram FST를 composition한 FST
	  (openfst를 사용해서 빌드한 결과)
	- korinvert.sym
	  kyfd의 input symbol 정의
	- worduniprob.sym
	  kyfd의 output symbol 정의
	- koreanuni.xml 
	  kyfd를 사용하기 위한 설정파일
	- kyfd
	  컴파일된 바이너리인데, 별도로 소스를 다운받고 설치해서 사용하자.
  ```

- 형태소분석
```
$ cd KFST/Rouzeta
$ foma
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

- 형태소분석용 FST를 save & load
```
...
foma[1]: save stack kor.stack
foma[1]: quit
$ foma
foma[0]: load stack kor.stack
foma[1]: up
apply up> 형태소분석은
...
```

- 커맨드라인에서 형태소 분석
```
$ flookup kor.stack
형태소분석은
형태소분석은   	형/nc태/nc소/nc분/nc석/nc은/nc
형태소분석은   	형/nc태/nc소/nc분/nc석/nc은/pt
형태소분석은   	형/nc태/nc소/nc분/nc석/nd은/nc
형태소분석은   	형/nc태/nc소/nc분/nc석/nd은/nr
....
```

- 태깅
```
$ cd KFST/Tagger
# 시스템에 설치한 kyfd를 사용한다.
$ rm kyfd
$ cat testme.txt | kyfd koreanuni.xml
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

# 문장을 캐릭터단위로 분리해서 공백구분자로 입력해야한다. 원래 공백문자는 <space>로 치환한다. 
```

- `SingleWordPhrase` 규칙을 보면 알겠지만, Rouzeta FST는 입력이 어휘형(자소단위,기호 등등)이고 출력이 표층형(어절 등)이다. 그런데, 형태소분석기는 이를 역으로 처리하는 프로그램이므로 실제 사용시에는 inverse 연산으로 FST를 뒤집어서 사용해야한다. `korfinaluni.fst` FST binary 파일은 이와 같이 inverse 연산한 결과와 unigram FST를 composition한 결과물이다. 어떻게 생겼는지 직접 보려면 `fstprint`를 사용해서 출력해보면 된다.
```
$ fstprint --isymbols=korinvert.sym --osymbols=worduniprob.sym korfinaluni.fst > korfinaluni.fst.txt
# fst의 마지막 필드는 weight인데, unigram FST를 composition하면서 들어온 값이다. 
$ more korfinaluni.fst.txt
...
572     3373    <epsilon>       /nc     2.7578125
572     3375    <epsilon>       /nr     1.61328125
573     4265    <epsilon>       /vb     6.78320312
573     3378    <epsilon>       /nc     2.7578125
573     3378    <epsilon>       /nr     1.61328125
574     13160   ·       ·       7.18847656
574     13161   가      가      4.54980469
574     13162   간      간      6.78320312
574     3696    거      거      6.27246094
574     13163   게      게      7.18847656
574     13164   경      경      6.49609375
...
133579	114136	길	기
133579	114137	긴	기
133579	114138	기	기
...

$ fstinfo korfinaluni.fst
fst type                                          vector
arc type                                          standard
input symbol table                                none
output symbol table                               none
# of states                                       230911
# of arcs                                         2993031
...
```

- `korfinaluni.fst`를 만드는 방법(해보지는 않았음)
```
# Rouzeta FST를 inverse 연산으로 뒤집은 다음 AT&T 포맷으로 저장한다. 
$ foma 
foma[0]: load stack kor.stack
foma[0]: invert net
foma[0]: write att > korinvert.fomaatt
...
$ more korinvert.fomaatt
...
1	113855	/ad	@0@
1	113342	/it	@0@
1	2	/vb	@0@
...
127409	66519	국	국
127409	98649	군	군
127409	66516	궁	궁
127409	104865	귀	귀
...
# AT&T 포맷을 openfst 포맷으로 변경하고 fst로 빌드하자(korfinal.fst)
# 세종코퍼스로부터 unigram FST를 추출해서 openfst 포맷으로 만든후 빌드한다(uni.fst)
# fstcompose를 이용해서 composition
$ fstcompose korfinal.fst uni.fst > korfinaluni.fst
```

- 바이그램 태거
  - [바이그램 한국어 품사 태거](https://shleekr.github.io/)
  - 지금까지 언급한 태거는 유니그램 확률만을 사용했지만, 바이그램 태거는 생성확률과 전이확률을 이용한다. fst의 사이즈는 좀 많이 커지지만(600M) 태깅 정확률은 좋아진다.
  - 사용법 
  ```
  # 앞서 기술한 유니그램 태거의 상용법과 동일하다.
  $ curl -OL https://shleekr.github.io/public/data/bitagger_aa
  $ curl -OL https://shleekr.github.io/public/data/bitagger_ab
  $ cat bitagger_aa bitagger_ab > bitaggerfile.tar.gz
  $ tar -zxvf bitaggerfile.tar.gz
  $ cd BiTagger/
  $ ls .
  koreanbi.xml  korinvert.sym  korinvertwordbifinal.fst  testme.txt  wordbiprob.sym
  # 시스템에 설치된 kyfd를 사용 
  $ rm kyfd
  $ cat testme.txt | kyfd koreanbi.xml
  --------------------------
  -- Started Kyfd Decoder --
  --------------------------
  Loaded configuration, initializing decoder...
  Loading fst korinvertwordbifinal.fst...
  Done initializing, took 6 seconds
  Decoding...
  나 /np 는 /pt <space> 학 교 /nc 에 서 /pa <space> 공 부 /na 하 /xv _ㅂ 니 다 /ef . /sf
  선 /nc 을 /po <space> 긋 /irrs /vb 어 /ex <space> 버 리 /vx 었 /ep 다 /ef . /sf
  고 맙 /irrb /vj 었 /ep 다 /ef . /sf
  ...
  ```

### Kyfd
- 입력 문자열을 linear FST로 만들고 이것과 Tagger FST(`korfinaluni.fst`)을 composition한 다음, begin -> end까지 shortest path를 찾으면, 그 path가 바로 tagging 결과가 된다. 이런 과정을 라이브러리로 구성해둔 decoder가 kyfd이다.  이 소스를 수정하면 좀더 편리한 API를 만들 수 있을 것 같다. 
  - kyfd를 fork해서 사용하기 편하게 수정한 버전, [kyfd](https://github.com/dsindex/kyfd)
  - 이것을 가지고 c, python interface를 개발,  [ckyfd](https://github.com/dsindex/ckyfd)
    - 형태소 분석결과를 정리해서 볼 수 있다.
	```
	$ cd ckyfd/wrapper/python
    $ python test_rouzeta.py -c koreanuni.xml
    Loading fst korfinaluni.fst...
    나는 학교에서 공부합니다.
    0	나는
    1	학교에서
    2	공부합니다.
    나	np	None	NP	0	0
    는	pt	None	JX	0	1
    학교	nc	None	NNG	1	2
    에서	pa	None	JKB	1	3
    공부	na	None	NNG	2	4
    하	xv	None	XSV	2	5
    _ㅂ니다	ef	None	EF	2	6
    .	sf	None	SF	2	7
    나는 답을 몰라.
    0	나는
    1	답을
    2	몰라.
    나	np	None	NP	0	0
    는	pt	None	JX	0	1
    답	nc	None	NNG	1	2
    을	po	None	JKO	1	3
    모르	vb	irrl	VV	2	4
    아	ec	None	EC	2	5
    .	sf	None	SF	2	6
	```

### Foma
- 영어에서 foma를 이용한 형태소분석기 만들기, [morpological analysis with FSTs](http://foma.sourceforge.net/dokuwiki/doku.php?id=wiki:morphtutorial)
  - 이것을 읽어 보면, Rouzeta에 있는 korean.lexc, morphrules.foma, kormoran.script 등을 더 잘 이해할 수 있을 것이다. 

### Problems on Rouzeta and Kyfd
- 미등록어
```
예) <space> 츠 카 <space> 그 룹 이 <space> 발 매 한 <space> ' 츠 카 <space> S ' <space> 라 는 <space> 이 름 의 <space> 실 버 폰 .

여기서 '츠 카'가 사전에 존재하지 않는 단어이므로 path가 존재하지 않아서 분석이 실패한다.( WARNING, no path found )
만약 입력문장을 단어별로 분리해서 형태소분석하고 그 결과물들을 붙여서 태깅을 하는 구조라면
중간에 어느 하나가 오분석되어도 큰 문제는 안될 것 같다. 하지만, 현재 모델에서는 one-path로 
형태소분석과 태깅을 동시에 수행하기 때문에 이런 문제가 생긴다.
현재 rouzeta는 이렇게 path를 찾지 못하는 경우가 매우 많기 때문에 어떤 방식으로 
개선해야 할 지 생각해봐야할 것 같다.
```
- 미등록 심벌
```
예) 가 벼 운 <space> 문 구 <space> 수 정 은 <space> 제 외 ) <space> ▲ <space> 기 본 <space> 질 문

여기서 '▲'는 unknown symbol인데, kyfd에서 "-unknown '<epsilon>'" 이렇게 지정해주면 아래와 같이 분석된다. 

가 볍 /irrb /vj _ㄴ /ed <space> 문 구 /nc <space> 수 정 /nc 은 /pt <space> 제 외 /nc ) /sr <space> 기 본 /nc <space> 질 문 /nc

unknown을 '<epsilon>'로 지정했기 때문에, 미등록 심벌은 무조건 사라지게 된다. 
unknown을 지정하지 않거나, 분석대상 문자열에 존재하는 심벌로 지정한 경우는 오류가 발생한다.

정확하게 처리하려면 rouzeta에 '<unk>' 심벌을 추가할 필요가 있어 보인다.
```
- 메모리 증가
```
kyfd에 보면 'reload' 옵션이 있는데, 예를 들어 '500'으로 설정하면 매 500 문장을 분석한 이후
fst를 다시 로딩하게 된다. 이것은 입력을 fst로 만들어서 compose할때마다 메모리가 늘어나기 때문에, 
적당한 시점에 이렇게 늘어난 메모리를 초기화 시켜주려고 만든 옵션인 것 같다.

"Reload the model after a certain number of sentences. Can be used
to flush the memory of expanded states for dynamically composed models 
that become unmanagably large after a time."

대량의 문서를 입력해서 메모리 증가를 모니터링한 결과 그렇게 큰 변화는 없었다. 
만약, 메모리가 꾸준히 증가한다면 kyfd를 그대로 서비스에 사용하기에는 적합하지 않을 것이다.
(메모리가 늘어나지 않도록 수정하거나 별도의 fst decoding 모듈을 개발해야 함)
```
- 분석 속도
```
적절한 길이의 문장단위로 입력했을 때, fst 기반 분석기의 속도는 다른 형태소분석기보다 빠를 것이다.
하지만, 문장의 길이가 더 길어지거나 입력이 문서인 경우는 문제가 된다. 문장이 길수록 탐색해야하는 composed fst의 크기도 
커지기 때문이다. 따라서, 문서로 입력되거나 문장의 길이가 매우 긴 경우를 처리하는 별도의 방법이 필요하다. 여기에는
문장분리기를 사용하거나, 윈도우를 이동시키며 분석하는 방법 등이 있을 수 있다. 이런 방법을 사용하면 미등록어로 인해 
분석이 실패할때도 해당하는 문장이나 어절만 제외하고 나머지를 분석할 수 있게 되는 효과도 얻을 수 있다.

또 하나의 문제는 매번 입력 문장을 linear fst로 만들고 composition한 다음, shortest path를 탐색한다는 데서 기인한다.
composition과 shortest path에 들어가는 비용은 생각보다 매우 크다. decoding까지 포함했을 경우, 일반 형태소분석기보다
분석 속도가 느린 경우도 많다. 따라서, composition 없이 단순히 fst를 탐색하면서 분석을 끝내는 방법이 필요하다. 
이렇게 되면 위 '메모리 증가'에서 언급된 문제도 사라질 것이다. 
```

