# firsthomework
>먼저 쉘크립트의 getopt와 getopts를 알아보자.

## getopt란

* getopt 명령은 예상되는 플래그와 인수를 지정하는 형식을 사용하여 토큰 리스트를 구문 분석한다. 플래그는 단일 ASCII 문자이며 뒤에 :이 올 경우 하나 이상의 탭 또는 공백으로 분리하거나 분리할 수 없는 인수가 있어야 합니다. 인수에는 복수 바이트를 포함시킬 수 있지만 플래그 문자로는 포함시킬 수 없습니다.
* getopt 명령은 모든 토큰을 읽은 후 또는 특수 토큰 -이 발생하는 경우 처리를 완료합니다. 그러면 getopt 명령은 처리된 플래그, - 및 남아 있는 토큰을 출력합니다.

* 토큰이 플래그와 일치하는 데 실패하는 경우 getopt 명령은 메시지를 표준 오류에 기록합니다.

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
![getopts](https://user-images.githubusercontent.com/94767794/142751115-cca30dd7-1d1f-42d5-bb5d-db12aa644e4f.png)

* 우선 getopts는 getopt의 bash버전이다. 그래서 이가 시스템명령과 구별됨을 명시하기 위해 s를 붙여 getopts로 적는것이다.
* getopts는 쉘에서 명령을 실해할 때 사용하는 옵션의 해석 작업을 쉽게 도와주는 명령이다.
  * 다만 getopts는 short옵션과 long옵션중 short옵션만을 처리할 수 있다.

## getopts의 작동 방식
