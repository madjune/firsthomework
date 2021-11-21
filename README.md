# firsthomework
>먼저 쉘크립트의 getopt와 getopts를 알아보자.

## getopt란

* **getopt** 명령은 예상되는 플래그와 인수를 지정하는 형식을 사용하여 토큰 리스트를 구문 분석한다. 플래그는 단일 ASCII 문자이며 뒤에 :이 올 경우 하나 이상의 탭 또는 공백으로 분리하거나 분리할 수 없는 인수가 있어야 합니다. 인수에는 복수 바이트를 포함시킬 수 있지만 플래그 문자로는 포함시킬 수 없습니다.
* **getopt** 명령은 모든 토큰을 읽은 후 또는 특수 토큰 -이 발생하는 경우 처리를 완료합니다. 그러면 **getopt** 명령은 처리된 플래그, - 및 남아 있는 토큰을 출력합니다.

* 토큰이 플래그와 일치하는 데 실패하는 경우 **getopt** 명령은 메시지를 표준 오류에 기록합니다.

```
#!/usr/bin/bsh
# parse command line into arguments
set -- `getopt a:bc $*`
# check result of parsing
if [ $? != 0 ]
then
        exit 1
fi
while [ $1 != -- ]
do
        case $1 in
        -a)     # set up the -a flag
                AFLG=1
                AARG=$2
                shift;;
        -b)     # set up the -b flag
                BFLG=1;;
        -c)     # set up the -c flag
                CFLG=1;;
        esac
        shift   # next flag
done
shift   # skip --
# now do the work
.
.
.
 ```
* 이런식으로 쉘 스크립트에서 사용해 옵션을 구문 분석한다


## getopts 란?

* 원본인 **getopt**에 있던 문제점들을 보완해 개선한 것
        * 인수의 공백이나 쉘 메타 문자 처리할 수 없었던 것과 오류 메시지 출력 비활성화 기능이 없던 것
* 위 상기한 문제점을 수정한 것 이외에도 GNU-스타일의 long 옵션과 그에대한 선택적인 인수를 구문 분석하는 기능 또한 도입되었다.
 
## getopt와 getopts 예를 통해 비교

```
getopt optstring [parameters]
getopts optstring varname [parameters]
```
* **getopt**와 **getopts**는 이런 형태로 쓰인다
```
-a   --article
-l   --language, --lang
-v   --verbose
```
* 아래 예에서 가능한 long arguments를 사용함


```
#!/bin/bash
VERBOSE=0
ARTICLE=''
LANG=en

# [EXAMPLE HERE]

if ​((VERBOSE > 2)); then
 ​printf '%s\n' 'Non-option arguments:'
 ​printf '%q ' "${remaining[@]]}"
fi

if ​((VERBOSE > 1)); then
 ​printf 'Downloading %s:%s\n' "$LANG" "$ARTICLE"
fi

if ​[[ ! $ARTICLE ]]; then  ​printf '%s\n' "No articles!">&2  ​exit 1
fi
>>-a   --article
-l   --language, --lang
-v   --verbose
save_webpage "https://${LANG}.wikipedia.org/wiki/${ARTICLE}"
```
* 위키디피아의 웹페이지를 다운로드하는 이런한 프로그램이 있다고 가정하자.
### getopt 사용
