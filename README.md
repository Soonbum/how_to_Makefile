# Makefile 사용법

[참조 자료](https://www.tuwlab.com/ece/27193)

리눅스 환경에서 소스 파일을 받아서 수동으로 프로그램을 설치해 본 분들은 다음 명령어가 익숙할 것입니다.

```
./configure
make
sudo make install
```

![빌드 예시](https://github.com/Soonbum/how_to_Makefile/assets/16474083/5cf44c36-87e9-4220-bfcd-9cc984757fe0)

Makefile 스크립트는 지정한 순서대로 빌드를 수행하는 데 사용합니다.
3개의 소스파일을 각각 컴파일하여 Object파일(*.o)을 생성하고, 이들을 한 데 묶는 링크 과정을 통해서 실행 파일인 app.out을 생성합니다. 여기서 foo와 bar에 정의된 함수를 main에서 호출하는 의존성이 존재합니다.

* Makefile이 없을 경우 빌드 과정 예시

```
gcc -c -o main.o main.c              # main.c 파일 컴파일하여 main.o 오브젝트 파일 생성
gcc -c -o foo.o foo.c                # foo.c, foo.h 파일 컴파일하여 foo.o 오브젝트 파일 생성
gcc -c -o bar.o bar.c                # bar.c, bar.h 파일 컴파일하여 bar.o 오브젝트 파일 생성
gcc -o app.out main.o foo.o bar.o    # 링커(ld)를 사용하여 오브젝트 파일들을 묶어서 실행파일(app.out) 생성
```

Makefile을 사용하면 Incremental Build, 즉 반복적인 빌드 과정에서 변경된 소스코드에 의존성(Dependency)이 있는 대상들만 추려서 다시 빌드하는 기능을 사용할 수 있습니다.
수동 빌드를 하지 않는 대신 실수를 줄여주고 변경사항만 빌드하므로 빌드 타임을 줄여줄 수 있다.

* Makefile 작성 튜토리얼

```
app.out: main.o foo.o bar.o
    gcc -o app.out main.o foo.o bar.o
 
main.o: foo.h bar.h main.c
    gcc -c -o main.o main.c
 
foo.o: foo.h foo.c
    gcc -c -o foo.o foo.c
 
bar.o: bar.h bar.c
    gcc -c -o bar.o bar.c
```

1. `make`: 모든 오브젝트 파일들이 만들어지고 실행파일(app.out)도 생성된다.
2. `make foo.o`: 대상 오브젝트 파일만 선별적으로 빌드한다.

* 빌드 규칙 블록
  - Target: 빌드 대상 이름. 통상 이 Rule에서 최종적으로 생성해내는 파일명을 써 줍니다.
  - Dependencies: 빌드 대상이 의존하는 Target이나 파일 목록. 여기에 나열된 대상들을 먼저 만들고 빌드 대상을 생성합니다.
  - Recipe: 빌드 대상을 생성하는 명령. 여러 줄로 작성할 수 있으며, 각 줄 시작에 반드시 Tab문자로 된 Indent가 있어야 합니다.

```
<Target>: <Dependencies>
    <Recipe>
```

* Makefile 변수 사용 예시

```
CC=gcc                     # 컴파일러
CFLAGS=-g -Wall            # 컴파일 옵션
OBJS=main.o foo.o bar.o    # 중간 산물 Object 파일 목록
TARGET=app.out             # 빌드 대상(실행 파일) 이름
 
$(TARGET): $(OBJS)
    $(CC) -o $@ $(OBJS)
 
main.o: foo.h bar.h main.c
foo.o: foo.h foo.c
bar.o: bar.h bar.c
```

`make -p`: 내부 정의된 변수를 확인할 수 있습니다.

* 자동 변수
  - $@: 현재 Target 이름
  - $^: 현재 Target이 의존하는 대상들의 전체 목록
  - $?: 현재 Target이 의존하는 대상들 중 변경된 것들의 목록
  - 그 외 자동 변수에 대한 정보는 [여기](http://www.gnu.org/software/make/manual/html_node/Automatic-Variables.html)를 참조하십시오.

* 클린 빌드

```
clean:
    rm -f *.o
    rm -f $(TARGET)
```

`make clean`: 잘못된 의존성 검사로 빌드 오류가 발생할 경우, 오브젝트 파일 등을 제거하고 다시 빌드 전 상태로 되돌려 놓습니다.

* Makefile 기본 패턴

```
CC=<C 컴파일러>
CFLAGS=<C 컴파일러 옵션>
LDFLAGS=<링크 옵션>
LDLIBS=<링크 라이브러리 목록>
OBJS=<Object 파일 목록>
TARGET=<빌드 대상 이름>
 
all: $(TARGET)
 
clean:
    rm -f *.o
    rm -f $(TARGET)
 
$(TARGET): $(OBJS)
$(CC) -o $@ $(OBJS)
```

전체 매뉴얼은 [여기](http://www.gnu.org/software/make/manual/make.html) 또는 [여기](https://makefiletutorial.com/)를 참조하십시오.

* 프로젝트 규모가 커질수록 Makefile을 관리하기 힘들어지므로 CMake 등과 같은 빌드 도구를 사용하는 것이 좋다.
