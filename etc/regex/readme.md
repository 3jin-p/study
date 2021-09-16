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
/s
. : ab와 줄바꿈 문자를 제외한 모든 문자 하와 매칭합니다.

```

### Bracket Expression
``` regexp
[abc] : [] 안에 있는 어느 문자와도 매칭합니다 a|b|c 와 동일합니다 (1)
[^abc] : [] 부정연산자 ^와 함께 사용하면 안에 있지 않은 어느 문자와도 매칭합니다 (2)
[a-c] : 범위지정 연산자 - 를 사용하면  a~c 까지의 어느 문자와도 매칭합니다. (3)
```