삶의 질을 높여주는 정규 표현식 정리
--

### Regular expressions 정규 표현식 이란?

문자열을 처리하는 형식 언어로 특정한 조건의 문자를 검색하거나 치환 하는데 매우 유용!

##### Anchor

``` regexp
^abc : 문자열 맨 앞의 abc (1)
abc$ : 문자열 맨 뒤의 abc (2)
^abc$ : abc와 정확히 일치하는 문자열 (3)
abc : 모든 문자열 abc (4)
```


*1. ^abc*
   - *abc*defasd  
   - asfasfabc  
   - *abc*abcabc

*2. abc$*
- asfas*abc*
- abcsabcwf
- abcabc*abc*

*3. ^abc$*
- *abc*
- abcsafqbc  
- abcabcabc

*4. abc*
- *abc*is*abc*
- asfaf*abc*afasf
- *abcabcabc*


#### Quantifiers

``` regexp
abc* ab와 0개 이상의 c를 포함한 문자열 (1)
abc+ ab와 1개 이상의 c를 포함한 문자열 (2)
abc? ab 와 0개 혹은 1개의 c릂 포함한 문자열 (3)
abc{3} ab 와 3개의 c를 포함한 문자열 (4)
abc{3,5} ab와 3~5개 의 c를 포함한 문자열 (5)
abc{3,} ab와 3개 이상의 c를 포함한 문자열 (6)
```

*1. abc**
- *abc* is alphabet
- *abc* is *ab*s
- *ababc*aaa*ab*aaa*abcccc*

*2. abc+*
- *abc* is alphabet
- *abc* is abs
- ab*abc*aaaabaaa*abcccc*

*3. abc?*
-  *abc* is alphabet
- *abc* is *ab*s
- ab*abc*aaa*ab*aaa*abc*ccc

*4. abc{3}*
-  abc is alphabet
- abc is abs
- ababcaaaabaaa*abccc*c

*5. abc{3,5}*
- abc is alphabet
- abc is abs
- *abccccc*abcaaabaaa*abcccc*

*6. abc{3,}*
- aabcc
- a*abccc*
- a*abcccccccccccccccc*

#### Character Classes
``` regexp
\d : 숫자 하나와 매칭합니다.    
\w : 글자 하나와 매칭합니다.
\s : 탭, 스페이스, 줄바꿈 등의 공백 문자 하나와 매칭합니다. 
\D : 숫자가 아닌 문자 하나와 매칭합니다.
\W : 글자가 아닌 문자 하나와 매칭합니다.
\S : 공백 문자가 아닌 문자 하나와 매칭합니다.
. : 줄바꿈 문자를 제외한 모든 문자 하나와 매칭합니다. 
```

*1. a\wc*
- *abc* asews abw *atc* *axc*qwe

*2. a\dc*
- *a2c*12abc4 *a2c*ww

*3. ^a\dc*

- *a2c*12abc4 a2cww

*4. a..*
- *asf*wfqf
- *asd*s*add*d*adw*da


### Bracket Expression
``` regexp
[abc] : [] 안에 있는 어느 문자와도 매칭합니다 a|b|c 와 동일합니다 (1)
[^abc] : [] 안의 ^는 부정연산자 역할을 합니다. 안에 있지 않은 어느 문자와도 매칭합니다 (2)
[a-c] : 범위지정 연산자 - 를 사용하면  a~c 까지의 어느 문자와도 매칭합니다. (3)
```

*1.[abc]*
- *abc*defghijklmnop *a*lph*ab*et *a*pple *b*e*a*r

*2.[^abc]*
- a*sd*bca*sds*a

*3.[a-c]*
- *a*e*b*e*c*d

### Grouping And Capturing

Capturing 은 문자열을 검색할 때 유용하게 사용됩니다.
``` regexp
a(bc)* : () 는 캡쳐 그룹을 생성합니다. 예제는 a와 bc 가 0회 이상 반복된 문자열을 찾습니다.
a(?:bc)* : (?:) 는 그룹을 생성하지만 캡쳐하지는 않습니다. () 와 동일한 문자열을 매칭하지만 조건절, Back Reference 등을 사용할 때, 캡쳐링 되지 않습니다.
a(?<named>bc) : (?<named>) 는 캡쳐 그룹의 이름을 지정합니다. 예제는 named 라는 캡쳐 그룹을 생성했습니다.
```

### Look ahead, behind

``` regexp
a(?=b) : b가 바로 뒤에있는 a 를 찾습니다. b 는 매칭되지 않습니다.
(?<=b)a : b가 바로 앞에있는 a 를 찾습니다. b 는 매칭되지 않습니다.

= 대신에 ! 를 사용하면 부정 연산자가 됩니다. 

a(?!b) : b가 바로 뒤에 없는 a 를 찾습니다. b 는 매칭되지 않습니다.
(?<!b) : b가 바로 뒤에 없는 a 를 찾습니다. b 는 매칭되지 않습니다.
```

### Back Reference 

Back Reference 는 캡쳐 그룹을 찾는 방법입니다.

``` regexp
([abc])(bcd)\숫자 : \숫자는 해당 캡쳐그룹과 동일한 패턴과 매칭됩니다 (1)
(?<named>[abc])\k<named> : \k<named> 는 해당 이름의 캡쳐 그룹과 매칭됩니다. (2)  
```

*1. (abc)(bcd)\2\1*
- *abcbcdbcdabc*
- abcdefghijkabc
- abab*abcbcdbcdabc*abab

*2. (?<named>abc)\k<named>*
- abababc
- ee*abcabc*ee
- abceeee*abcabc*eeabc

### Flags

정규식은 일반적으로 \ \ 사이에 작성합니다. 
닫는 \ 뒤에 Flag 를 지정할 수 있습니다.

``` regexp
g(global): 문자열에서 매칭되는 모든 항목을 찾습니다
m(multi line): 모든 문자열에서 매칭하는 것이 아닌 Line 별로 매칭합니다. ^, $ 를 사용할 때 유용합니다.
i(insensitive): 대소문자 구분을 무시하고 매칭합니다.
```

### Useful Regex

실무에서 작업하면서 사용한 정규식을 나중을 위해 정리해둡니다.

``` regexp
- 휴대폰 번호
^01([0|1|6|7|8|9]?)([0-9]{3,4})([0-9]{4})$

- 주민등록번호
\\d{2}([0]\\d|[1][0-2])([0][1-9]|[1-2]\\d|[3][0-1])[1-4]\\d{6}

- 특수문자 제한 (알파벳과 한글 숫자, 언더바 와 하이픈만 허용)
^[a-zA-Z0-9_\\-ㄱ-ㅎㅏ-ㅣ가-힣]*$
```
