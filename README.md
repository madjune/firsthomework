# getopt와 getopts를 알아보자.

## getopt란

* **getopt** 명령은 예상되는 플래그와 인수를 지정하는 형식을 사용하여 토큰 리스트를 구문 분석한다. 플래그는 단일 ASCII 문자이며 뒤에 :이 올 경우 하나 이상의 탭 또는 공백으로 분리하거나 분리할 수 없는 인수가 있어야 한다. 인수에는 복수 바이트를 포함시킬 수 있지만 플래그 문자로는 포함시킬 수 없다.
* **getopt** 명령은 모든 토큰을 읽은 후 또는 특수 토큰 -이 발생하는 경우 처리를 완료한다. 그러면 **getopt** 명령은 처리된 플래그, - 및 남아 있는 토큰을 출력힌다.

* 토큰이 플래그와 일치하는 데 실패하는 경우 **getopt** 명령은 메시지를 표준 오류에 기록다.

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
```
# parse everything; if it fails we bail
args=`getopt 'a:l:v' $*` || exit
# now we have the sanitized args... replace the original with it
set -- $args

while true; do
    case $1 in
      (-v)   ((VERBOSE++));  shift;;
      (-a)   ARTICLE=$2; shift 2;;
      (-l)   LANG=$2; shift 2;;
      (--)   shift; break;;
      (*)    exit 1;;           # error
    esac
done

remaining=("$@")
```
* 위 getopt는 선택적 인수를 지원하지않는다. 또한 article 제목에 공백이나 ?등의 특수기호가 들어간다면 바로 정지될 것이다

### getopts 사용
```
#!/bin/sh
while getopts ':a:l:v' opt; do
    case $opt in
      (v)   ((VERBOSE++));;
      (a)   ARTICLE=$OPTARG;;
      (l)   LANG=$OPTARG;;
      (:)   # "optional arguments" (missing option-argument handling)
            case $OPTARG in
              (a) exit 1;; # error, according to our syntax
              (l) :;;      # acceptable but does nothing
            esac;;
    esac
done

shift "$OPTIND"
# remaining is "$@"
```
* 더이상 쉘스크립트 옵션을 바로 작동시키지 않아서 그것들을 시프트 할필요는 없어졌다. 다만 나머지 인수를 얻기위해 slicing 작업이 필요해졌다.

# sed 명령어

## sed란?
* Stream Editor 의 약자로 원본 텍스트 파일을 편집하는 명령어이다.
### vi편집기와의 차이
* sed는 명령어 형태로 편집이 되어 vi처럼 실시간 편집이 아니다.
* 원본을 건드리지 않고 편집하기 때문에 작업이 완료되기 떄문에 기본적으로는 원본에는 영향을 주지 않는다.

## sed 개념

![img1 daumcdn](https://user-images.githubusercontent.com/94767794/142754171-570bf474-9e59-44c2-b842-a7ac84eecd46.png)

* sed는 내부적으로 특수한 저장 공간인 두 가지의 버퍼를 이용한다. 
        * **패턴 버퍼(패턴 스페이스)**
        * **홀드 버퍼(홀드 스페이이스)**
* 위 그림에서 보이듯 sed는 먼저 InputStream으로 파일의 내용을 가져 온다.
* 패턴 버퍼에 내용을 담고 있다가 데이터의 변형과 추가를 위해 다시 사용하는 것이 **홀드 버퍼**이다
* 그리고 작업이 완료되면 패턴 버퍼에 내용이 담기고 그를 OutputStream에 보냈으때 우리가 원하는 결과가 출력된다.

## sed 사용법

```
sed -n -e 'command' [input file]
```
* sed는 기본적으로 위와 같은 형태로 사용한다
        * **-n** : 이를 이용하면 pattern buffer의 내용을 자동 출력하지 않게함.
        * 너무 많은 데이터의 출력을 막기위해 **기본적으로 sed -n을 sed의 기본 형태라고 생가하면 된다.**
        * **-e** : 이를 이용해 command를 가지고 텍스트 파일을 가공해 준다.

|command||설명|
|:---:|:---:|:-----|
|-p|print|특정 행을 출력|
|-d|delete|특정 행 삭제|
|-s|substitute|단어 치환|
|-a, -i|append, insert|문자열 추가|
|-c|change|특정 행의 내용을 전부 교체|
|-r|read|특정 행에 파일의 내용을 추가|


# awk 명령어

## awk란?
* Aho+Weinberger+Kernighan 다른 명령어와는 다르게 이름을 디자인한 사람들의 이니셜을 조합해서 따왔다.
* **awk**는 파일로부터 레코드를 선택하고 선택된 레코드에 포함된 값을 조작하거나 데이터화하는 것을 목적으로 사용하는 프로그램입니다.
1) awk 명령어의 입력으로 지정 파일로부터 데이터 분류
2) 분류된 텍스트 데이터를 바탕으로 패턴 매칭여부 검사나 데이터 조작 및 연산 등의 액션수행
3) 그 결과를 출력
* 위와 같은 기능을 수행한다

### 레코드?
![998C54465D1CBC7B16](https://user-images.githubusercontent.com/94767794/142755129-6b7d96ec-6728-4c20-b376-dbc9ba57b6a1.png)
* 기본적으로 위처럼 입력 데이터를 레코드로 인식한다.
* 그리고 각 레코드에 들어있는 텍스트를 공백문자로 구분된 필드로 분류한다
## awk 명령의 기본 형식과 옵션, 그리고 사용 예
```
    awk [OPTION...] [awk program] [ARGUMENT...]
      OPTION
        -F        : 필드 구분 문자 지정.
        -f        : awk program 파일 경로 지정.
        -v        : awk program에서 사용될 특정 variable값 지정.
      awk program
        -f 옵션이 사용되지 않은 경우, awk가 실행할 awk program 코드 지정.
      ARGUMENT
        입력 파일 지정 또는 variable 값 지정.
```
* awk 명령어의 기본 형식과 옵션은 위와 같다.
```
pattern { action }
```
```
awk [OPTION...] 'pattern { action }' [ARGUMENT...]
```
* **awk program** 의 구조와 사용할때 형태는 위처럼 되는데 **pattern 과 action은 모두 생략이 가능하다**
        * 생략 될 때 pattern의 경우 모든 레코드가 적용되고 action의 경우 print가 적용된다.
![99C761465D1CBF9B28](https://user-images.githubusercontent.com/94767794/142755114-d3296550-ecbf-48f0-a08b-e7627b202ea3.png)
* 위 그림은 awk로 할 수 있는 작업의 간단한 예시 이다.


|awk 사용 예|명령어 옵션|
|:---|:-----|
|파일의 전체 내용 출력|	awk '{ print }' [FILE]|
|필드 값 출력|awk '{ print $1 }' [FILE]|
|필드 값에 임의 문자열을 같이 출력|awk '{print "STR"$1, "STR"$2}' [FILE]|
|지정된 문자열을 포함하는 레코드만 출력	awk '/STR/' [FILE]|
|특정 필드 값 비교를 통해 선택된 레코드만 출력|awk '$1 == 10 { print $2 }' [FILE]|
|특정 필드들의 합 구하기|awk '{sum += $3} END { print sum }' [FILE]|
|여러 필드들의 합 구하기|awk '{ for (i=2; i<=NF; i++) total += $i }; END { print "TOTAL : "total }' [FILE]|
|레코드 단위로 필드 합 및 평균 값 구하기|awk '{ sum = 0 } {sum += ($3+$4+$5) } { print $0, sum, sum/3 }' [FILE]|
|필드에 연산을 수행한 결과 출력하기|awk '{print $1, $2, $3+2, $4, $5}' [FILE]|
|레코드 또는 필드의 문자열 길이 검사|awk ' length($0) > 20' [FILE]|
|파일에 저장된 awk program 실행|awk -f [AWK FILE] [FILE]|
|필드 구분 문자 변경하기|awk -F ':' '{ print $1 }' [FILE]|
|awk 실행 결과 레코드 정렬하기|awk '{ print $0 }' [FILE]|
|특정 레코드만 출력하기|awk 'NR == 2 { print $0; exit }' [FILE]|
|출력 필드 너비 지정하기|awk '{ printf "%-3s %-8s %-4s %-4s %-4s\n", $1, $2, $3, $4, $5}' [FILE]|
|필드 중 최대 값 출력|awk '{max = 0; for (i=3; i<NF; i++) max = ($i > max) ? $i : max ; print max}' [FILE]|


>>아이고 끝났다 혹시몰라서 commit 좀더 해두겠습니다
