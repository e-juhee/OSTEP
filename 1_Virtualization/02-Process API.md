# 1. fork()

- 프로세스를 생성하는 시스템 콜

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main(int argc, char *argv[])
{
    printf("hello world (pid:%d)\n", (int) getpid());

    int rc = fork(); // rc = return code

    if (rc < 0) { // fork 실패한 경우
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) { // 자식일 경우
        printf("hello, I am child (pid:%d)\n", (int) getpid());
    } else { // 부모일 경우
        printf("hello, I am parent of %d (pid:%d)\n", rc, (int) getpid());
    }
    return 0;
}
```

```bash
prompt> ./p1
hello world (pid:29146)
hello , I am parent of 29147 (pid:29146)
hello , I am child (pid:29147)
```

- PID = process identifier = 프로세스 식별자
  - 프로세스의 실행이나 중단과 같이 특정 프로세스를 대상으로 작업해야 할 경우 프로세스를 지칭하기 위해 사용된다.
- 생성된 자식 프로세스는 호출한 부모 프로세스의 거의 정확한 복사본이다.
  - 즉, OS 입장에서는 이제 프로그램 p1이 두 개가 실행 중이며, 둘 다 fork() 호출로부터 반환될 준비가 된 것으로 보이게 된다.
- hello world가 한 번만 출력된 것을 보면 자식 프로세스는 main() 함수의 첫 부분부터 시작하지 않고, 마치 자기가 fork()를 호출한 것처럼 시작한다.
- 자식 프로세스는 자신의 주소 공간, 레지스터, PC 값을 가지고, fork()의 반환 값도 부모와 다르다.
  - 부모 프로세스는 fork()를 호출해서 자식의 PID를 반환받고, 자식 프로세스는 0을 반환받는다.
  - 이 반환값의 차이를 활용해서 위 코드에서와 같이 부모와 자식 프로세스가 서로 다른 코드를 실행하게 프로그램을 작성할 수 있다.
- 위 프로그램의 출력은 nondeterministic하다. (비결정적이다.)
  - 자식이 생기면서 활성 프로세스가 두 개가 되면 어떤 프로세스가 실행될지를 CPU 스케줄러가 결정한다.

# 2. wait()

- 부모 프로세스가 자식 프로세스의 종료를 대기해야 하는 경우 사용하는 시스템 콜

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

int main(int argc, char *argv[])
{
    printf("hello world (pid:%d)\n", (int) getpid());
    int rc = fork();
    if (rc < 0) { // fork 실패
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) { // 자식 프로세스
        printf("hello, I am child (pid:%d)\n", (int) getpid());
    } else { // 부모 프로세스
        int wc = wait(NULL);
        printf("hello, I am parent of %d (wc:%d) (pid:%d)\n",
               rc, wc, (int) getpid());
    }
    return 0;
}
```

```bash
prompt> ./p2
hello world (pid:29266)
hello , I am child (pid:29267)
hello , I am parent of 29267 (wc:29267) (pid:29266)
```

- 위 코드에서 부모 프로세스가 wait()을 호출해서 자식 프로세스 종료 시점까지 자신의 실행을 잠시 중지시킨다.
- 자식 프로세스가 종료되면 wait()가 리턴한다.
- 이렇게 wait()를 위 코드와 같이 추가하면 프로그램은 항상 동일한 결과를 출력한다.
  - 자식이 먼저 실행되는 경우 당연히 자식 프로세스가 먼저 출력됨
  - 부모가 먼저 실행되는 경우 wait()를 호출하기 때문에 자식이 먼저 실행됨

# 3. exec()

- 자신의 복사본이 아닌 다른 프로그램을 실행하는 시스템 콜

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <sys/wait.h>

int main(int argc, char *argv[])
{
    printf("hello world (pid:%d)\n", (int) getpid());
    int rc = fork();
    if (rc < 0) { // fork 실패
        fprintf(stderr, "fork failed\n");
        exit(1);
    } else if (rc == 0) { // 자식 프로세스
        printf("hello, I am child (pid:%d)\n", (int) getpid());
        char *myargs[3];
        myargs[0] = strdup("wc");      // 실행할 프로그램: "wc"
        myargs[1] = strdup("p3.c");    // 프로그램의 인자: 파일 이름 "p3.c"
        myargs[2] = NULL;              // 인자 목록의 끝을 표시하기 위한 null
        execvp(myargs[0], myargs);     // "wc" 명령 실행
        printf("this shouldn't print out\n"); // execvp가 성공하면 이 코드는 실행되지 않음
    } else { // 부모 프로세스
        int wc = wait(NULL);
        printf("hello, I am parent of %d (wc:%d) (pid:%d)\n",
               rc, wc, (int) getpid());
    }
    return 0;
}
```

```bash
prompt> ./p3
hello world (pid:29383)
hello , I am child (pid:29384)
29 107 1030 p3.c
hello , I am parent of 29384 (wc:29384) (pid:29383)
```

- 위 예제에서는 자식 프로세스가 wc 프로그램을 실행하기 위해 execvp()를 호출한다.
  - exec()는 6가지 변형이 존재한다: execl(), execle(), execlp(), execv(), execvp(), execve()
- 실행할 파일의 이름과 인자를 전달하며 호출한다.
- 실행 파일의 코드와 정적 데이터를 load해서 현재 실행 중인 프로세스의 코드 세그먼트와 정적 데이터 부분을 덮어 쓴다.
  - 힙, 스택, 메모리 공간의 다른 부분들도 새롭게 초기화되며 이전에 실행되던 p3의 흔적은 사라지게 된다.
- 그 후 프로세스의 argv와 같은 인자를 전달하여 프로그램을 실행시킨다.
- 새로운 프로세스를 생성하는 것이 아니라, 현재 실행 중인 프로그램 p3을 다른 실행 프로그램인 wc로 대체하는 것이다.
- exec()를 성공적으로 호출하면 반환하지 않고, 마치 p3.c가 실행된 적이 없었던 것처럼 된다.

# 4. 왜 fork()와 exec()를 분리했을까?

- 새 프로세스를 생성하는 단순한 작업을 위해 fork()와 exec()를 분리해둔 이유가 뭘까?
  - 이것은 UNIX 쉘을 구축하는 데 매우 중요한 역할을 한다.
- fork()와 exec를 분리하면, 쉘은 fork() 호출 후와 exec() 호출 전에 코드를 실행할 수 있게 된다.
  - 이 시점에 쉘이 새롭게 실행될 프로그램의 환경을 변경하거나 설정할 수 있으며, 이를 통해 다양한 유용한 기능을 쉽게 구현할 수 있다.
- Shell = 쉘
  - 단순히 하나의 사용자 프로그램이다.
  - 쉘의 종류: tcsh, bash, zsh 등
- 쉘의 작동 방식
  - (1) 사용자에게 프롬프트를 보여주고, 사용자 입력을 기다린다.
  - (2) 사용자가 명령어(실행 파일의 이름과 인자)를 입력한다.
  - (3) 파일 시스템에서 실행 파일의 위치를 찾아낸다.
  - (4) fork()를 호출해 명령어를 실행할 자식 프로세스를 만든다.
  - (5) 자식이 exec()를 호출해 명령어를 실행한다.
  - (6) 쉘이 wait()를 호출해 자식 프로세스가 종료되기를 기다린다.
  - (7) 자식이 종료되면 wait()에서 반환되고, 쉘은 다시 프롬프트를 출력하며 다음 명령어를 기다린다.
- fork()와 exec() 분리의 장점
  - 쉘이 다양한 유용한 작업을 매우 쉽게 수행할 수 있도록 해준다. 예를 들어:
    ```bash
    prompt> wc p3.c > newfile.txt
    ```
    - wc의 출력을 newfile.txt로 재지정한다. (`>`: 리다이렉션)
    - 쉘이 이 작업을 수행하는 방식은 매우 간단하다. 자식이 생성되고 exec()를 호출하기 전에 표준 출력을 닫고 newfile.txt를 열어 두면, 실행될 프로그램인 wc의 출력은 화면이 아닌 해당 파일로 전달된다.
- 입출력을 리다이렉트하는 프로그램 예시

  ```c
  #include <stdio.h>
  #include <stdlib.h>
  #include <unistd.h>
  #include <string.h>
  #include <fcntl.h>
  #include <sys/wait.h>

  int main(int argc, char \*argv[]) {
  int rc = fork();
  if (rc < 0) { // fork 실패
    fprintf(stderr, "fork failed\n");
    exit(1);
  } else if (rc == 0) { // 자식 프로세스
    // 표준 출력 닫기
    close(STDOUT_FILENO);
    // 출력 파일 열기 및 연결
    open("./p4.output", O_CREAT | O_WRONLY | O_TRUNC, S_IRWXU);

    // exec "wc"
    char *myargs[3];
    myargs[0] = strdup("wc");
    myargs[1] = strdup("p4.c");
    myargs[2] = NULL;
    execvp(myargs[0], myargs);
  } else { // 부모 프로세스
    int wc = wait(NULL); // 자식 프로세스 종료 대기
  }
  return 0;

  }
  ```

  - UNIX는 open()을 호출할 때, 사용되지 않은 가장 낮은 번호의 파일 디스크립터가 새로 연 파일에 할당된다.
  - 1번인 STDOUT_FILENO을 close했으므로, 1번이 사용 가능한 파일 디스크립터가 되어서, 1번에 새로 열린 파일 p4.output이 할당된다.
  - 따라서, 자식 프로세스가 printf()와 같은 루틴을 통해 표준 출력 파일 디스크립터에 쓰기를 하면, 그 출력이 파일 p4.output에 전달된다.

- UNIX 파이프도 비슷한 방식으로 구현된다.
  - (하지만, pipe() 시스템 콜을 사용한다.)
  - 한 프로세스의 출력이 커널 내 파이프(큐)에 연결되고, 다른 프로세스의 입력도 동일한 파이프에 연결된다.
  - 따라서 한 프로세스의 출력이 자연스럽게 다음 프로세스의 입력으로 사용된다.
  - 이를 통해 긴 명령어 체인을 만들 수 있다.
  - ex) 파일에서 특정 단어를 찾고 그 단어가 나타난 횟수를 세는 명령어
    ```bash
    grep -o foo file | wc -l
    ```

# 5. 프로세스 제어와 유저

- UNIX 시스템에는 fork(), exec(), wait() 외에도 프로세스와 상호작용하기 위한 다양한 인터페이스가 존재한다.
- 예를 들어, kill() 시스템 콜은 프로세스에 signal을 보내는 데 사용되어 프로세스를 일시 중지하거나 종료하는 등 유용한 명령을 전달할 수 있다.
- 대부분의 UNIX 쉘에서는 특정 키 조합을 사용해 현재 실행 중인 프로세스에 특정 신호를 보낼 수 있다.
  - Ctrl+C: SIGINT(인터럽트) 신호를 보내 프로세스를 종료한다.
  - Ctrl+Z: SIGTSTP(중지) 신호를 보내 프로세스를 일시 중지한다.
