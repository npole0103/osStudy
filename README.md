# osStudy
Operation System Study

**교수님의 코드 공개 불허로 인해 과제 코드는 업로드 하지 않음.**

---

## HW1 
### FAT Table 구현

FAT32는 디스크에 저장됨.
DevReadBlock / DevWriteBlock을 바탕으로 각 함수들이 동작

FAT 엔트리 갯수 = Block Size / 4bytes

만약 블럭 사이즈가 16이라면 엔트리 갯수는 4

FAT 엔트리 값이 0이면 비어있음, -1이면 마지막 블록임을 나타냄.

`void FatInit()` : FAT table을 0으로 채워서 초기화 하는 함수

`void FaTAdd(int lastBlkNum , int newBlkNum)` : lastBlkNum 의 다음 블록 을 newBlkNum 으로 지정함. lastBlkNum 이 1 이면 빈 파일에 logical Block 0 을 추가한다고 가정.

`int FatGetBlockNum(int firstBlock , int LogicalBlkNum)` : firstBlock 에서 LogicalBlkNum 의 physical block number를 획득

`int FatRemove(int firstBlock , int startBlock)` : startBlock 부터 마지막 블록까지 제거. firstBlock은 파일 시작을 지정. 리턴 값은 제거된 블록 갯수

---

## HW2
### Buffer Cache 구현

TAILQ(queue.h)를 이용한 버퍼 캐쉬 구현.

Buffer List : 버퍼들이 순차적으로 연결되어 있는 리스트

Clean List : 변경되지 않은 블럭들이 연결되어 있는 리스트

Dirty List : 변경된(새로운 내용이 기록된) 블럭들이 연결되어 있는 리스트

LRU(Least Recently Used) List : 가장 최근에 접근된 블럭이 TAIL 부분에 들어가고, 가장 나중에 접근된 블럭이 HEAD 부분에 위치


`Void BufInit(void)` : TAILQ_INIT 필요

`Void BufRead(int blkno, char* pData)`  
blkno : 읽을 block의 번호.  
pData : 블록을 담을 메모리 공간. 버퍼에 존재하는 데이터를 pData로 읽어옴.  

`Void BufWrite(int blkno, char* pData)`  
blkno : 저장할 block 번호.  
pData : 저장할 블록을 담을 메모리 공간. 데이터를 버퍼상에 저장함.

`Void BufSync(void)`  
Dirty list에 연결된 buffer의 블록들을 디스크로 저장한다.  
단, HEAD부터 buffer는 더티리스트에서 클린리스트로 넘긴 후 TAIL 쪽으로 이동한다.

`Buf* BufGet(int blkno)`  
blkno의 블록을 가지는 버퍼를 검색해서 해당 버퍼의 주소 값을 반환. 버퍼가 없다면 NULL 반환

`void BufSyncBlock(int blkno)`  
blkno의 블록을 가지는 버퍼가 dirty block이라고 가정함.
해당 버퍼를 디스크로 Sync하는 작업. BufSync()와 동일한 동작이지만, 특정 블록만 디스크로 Sync함.

---

## HW3
### File System 구현

Buffer Cache와 FAT table을 이용한 파일 시스템

0번 블럭에는 File System Info(SuperBlock)이 저장됨.

1 ~ 129번 블럭에는 FAT Table이 존재함.

130번 블럭부터 할당 가능. 130번 블럭은 Root Block

0 SuperBlock / 1 ~ 129 Fat table Region / 130 ~ 4096 Data Region

File Descriptor Table : 사용 중인 엔트리 갯수와 DescEntry(해당 엔트리가 사용중인지, File table의 포인터 값)가 존재

File Table : 사용 중인 엔트리 갯수와 File struct(해당 파일이 사용 중인지, flg, blkno, entryindex, fileOffset) 존재


`void Format()` : 파일 시스템을 포맷하고 초기화 하는 동작 수행

`void Mount()` : 파일 시스템을 포맷이 아닌, 전원을 끄기 전의 파일시스템을 사용하는 동작이다.

`void Unmount()` : 전원을 끌 때 호출되는 함수라고 간주한다. 버퍼 캐쉬에 저장된 Dirty block들을 디스크로 저장한 후에, 가상 디스크를 close한다. 이때 `BufSync()`사용

`int MakeDirectory(char* path, int mode)` : 디렉토리를 생성하는 함수

`int RemoveDirectory(char* path)` : 해당 디렉토리를 삭제하는 함수. 단, 빈 디렉토리만 삭제 가능함. 디렉토리 내에 파일이 있으면 삭제 불가능함.

`Directory* OpenDirectory(char* name)` : 디렉토리를 Open하는 함수. 리눅스 opendir() 함수와 동일함. 성공하면 Directory 주소 반환, 실패하면 NULL 반환.

`FileInfo* ReadDirectory(Directory* pDir)` : 디렉토리에 저장된 파일 정보를 획득함. readdir() 동작과 동일. pDir에 더 이상 획득한 파일 정보가 없으면, NULL 반환.

`int CloseDirectory(Directory* pDir)` : 열린 디렉토리를 닫는 함수. 리눅스의 closedir() 동작과 동일함.

`int OpenFile (char* szFileName, OpenFlag flag)` : 파일을 Open 하는 함수. 만약 파일이 존재하지 않는다면 파일 생성.

`int WriteFile(fd, pBuf, BLOCK_SIZE)` : 해당 fd에 pBuf에 있는 내용을 쓰는 함수.

`int ReadFile(fd, pBuf, BLOCK_SIZE)` : 해당 fd에 저장된 내용을 pBuf에 읽어들이는 함수.

`int CloseFile(int fd)` : 해당 fd에 해당하는 파일을 닫는 함수.

`int RemoveFile(char* path)` : 해당 경로에 해당하는 파일을 삭제하는 함수.


---

## 운영체제 개념 정리

---

**컴퓨터 시스템 : 4개의 계층으로 구성되어 있음**  
1. 사용자
2. 시스템, 응용프로그램
3. OS 하드웨어를 제어하는 큰 소프트웨어
4. 하드웨어 CPU, DRAM, IO(DISK, KEYBOARD)

---

**OS가 존재하지 않는다면**  
- 실행파일을 실행시켜주는데 제일 중요한 역할을 하는데 DRAM CPU 등을 연결하기가 어렵다
- 모니터란 어떤 디바이스가 있는데 이것을 출력하려면 시스템 소프트웨어 펌웨어가 필요 이것을 OS가 해결해줌.

---

**OS의 목적**  
1. 사용자의 프로그램을 실행시켜주는 것
2. 사용자가 컴퓨터 시스템을 쉽게 사용할 수 있도록 해주는 것.
3. 프로그램 개발자에게 System call을 사용하게 해서 쉽게 접근할 수 있도록 해줌
4. CPU라던지 메모리를 효율적으로 사용할 수 있게 해주는 것.

---

**OS의 정의**  
사용자와 하드웨어에서 존재하는 아주 큰 프로그램  
CPU 메모리 등을 제어하고 할당 해주는 소프트웨어를 OS라고 칭함  
항상 돌아가고 있는 것이 OS임 다른 말로 Kernel이라고도 함.  
OS를 판매하는 회사가 OS의 범위를 정의하게 됨. 우리가 정의할 순 없다.  

---

**OS 구조**  
프로세스 매니지먼트 : 프로세스 생성하고 정렬하고 정지하게 관리해주고, 여러 개의 프로세스가 동작하도록 CPU에서 스케쥴링 해줌.

메모리 매니지먼트 : Virtual memory 가상메모리(무한한 공간)를 제어함. DRAM 공간을 효율적으로 사용할 수 있도록 해줌. 프로세스가 사용하던 메모리를 제거해준다던지. Swap space : Disk 일부 공간을 이용해서 DRAM보다 큰 메모리를 저장할 수 있도록 해줌.

파일 매니지먼트 : 데이터를 파일로 관리해주는 역할. 이름이 있는 파일로 관리해줌. 디렉토리로 그룹핑해서 파일을 관리하게 해줌. 파일 삭제, 생성 정보관리, 디렉토리 생성, 삭제 등등 전부 파일 매니지먼트에서 관리해줌.

네트워크 매니지먼트 : 외부 원격이 없는 컴퓨터와 데이터를 통신하게 해주는 시스템. 컴퓨터 네트워크에서 배움.

I/O 매니지먼트 : keyboard, disk, monitor 등등을 관리해주는 매니지먼트. 우리 수업에서는 disk 매니지먼트만 다룸.

---

**USER모드에서 KERNEL 모드로 들어가는 방법 4가지**  
1. USER모드에서 시스템 콜을 호출하면 됨. EX) fork
2. Interrupts 발생하게 되면, 컨트롤+c를 누르면 키보드 인터럽트가 걸리게 되고 CPU는 키보드 처리를 하기 위해서 커널모드로 들어가게 됨.
3. 0으로 나누는 경우 프로그램을 종료하기 위해 Exceptions이 발생하고, USER모드에서 돌다가 KERNEL 모드로 돌아가고, 강제 종료 시킴.
4. APP이 동시에 4개가 돌아가는 프로그램(0, 1, 2, 3번 앱이 있음)이 있다고 가정할 때, USER모드에서 돌다가 0번에서 1번앱으로 넘어가는 순간 KERNEL모드에 있는 스케쥴러가 관여하면서 다른 프로세스를 실행하게 됨. 

개발자가 합법적으로 KERNEL모드로 들어가는 방법은 시스템 콜 호출밖에 없다

---

**CPU는 mode bit라는 것을 가지고 있음. mode bit = 1**  
EX)
1. 우리가 짠 코드 중에 fork가 있다면
2. 시스템 콜을 호출하고
3. CPU는 커널 코드를 실행하기 위해 커널 모드로 들어가게 됨. mode bit = 0으로 바꿈
4. fork에 대한 것을 처리하고 난 뒤 mode bit = 1을 다시 반환
5. 시스템 콜을 리턴함.

---

**운영체제의 구조**  
운영체제가 사용자에게 제공하는 인터페이스
- CLI(Command Interpreter) : 터미널에서 명령을 받아서 처리함.
- Desktop metaphor Interface : 윈도우 10 생각하면 됨.

---

**System Call and API**  
시스템 콜 : OS 서비스를 사용할 수 있다. 대부분 C언어로 많이 구현되어있다.

API는 미들웨어가 제어하는 인터페이스이다. OS -> Middle-Ware -> APP
POSIX API, Java API가 Middle-Ware 프로그램

---

**OS 구조**  
Layered Approach : 각 레이어가 밑에 층에 있는 함수들 하나하나 씩 출력하게 됨

모노리딕 구조 / 간단한 구조 : 구조화 되지 않은 구조. 우리가 지양해야함.

Microkernel System Structure : 커널모드 Microkernel 필요한 최소한의 기능만 추가하고 나머진 다 유저모드로 올려버림

1. 프로세스 매니지먼트
2. 메모리 매니지먼트
3. 싱크로나이제이션(동기화)
4. IPC

Modules 모듈 : 대부분의 OS에서 사용되고 있음

Solaris Modular Approach : 디바이스 드라이버 외에도 파일 시스템에서 사용됨. 하위에 NTFS라는 것이 존재함.

---

**Virtual Machines**  
Hardware와 OS를 하나로해서 새로운 하드웨어처럼 보여주게끔 해주는 것.

---

**System Boot 과정**  
컴퓨터를 시작해서 커널을 로딩하는 과정

One-step Bootstrap (BIOS)  
1. DRAM, CPU 등 기타 디바이스 상태를 검사함.
2. 문제가 없다면 커널을 메모리 상에 올리고 시작을 하게 됨.

Two-step booting process (Linux, Unix and Windows OS)  
1. Bootstrap program 바이오스가 구동되면
- 1. 하드웨어 검사
- 2. 중요 Mast Boot Record(MBR)을 메모리(DRAM) 상에 로드함
- 3. boot block 코드를 디스크에서 로드하게 됨. 커널을 메모리상에 올림
2.  boot block 코드가 OS에서 로드되고 실행됨

부트 블록이 있으면 좋은 점  
1. 부트 블록 안에 OS를 선택할 수 있는 기능이 있음.  
2. 부트 블록이 있다면 다양한 OS를 선택할 수 있음  

---

**Secondary-Storage-Structure**  
Platter(플래터) : Spindle 기준으로 Platter가 존재함.

Sector : 디스크 중 하나의 부분.

Track : 동일 반경에 모든 sector들.

Cylinder : 동일 반경 상에 존재하는 Track들의 묶음.

ARM : 플래터를 읽은 작은 바늘

TOP : 플래터 윗면

BOTTOM : 플래터 밑면

---

**플래터가 1개일 때**  
TOP 섹터 0번부터 시작해서 다음 섹터는 동일 반경에 BOTTOM ->
그 다음 작은 부분의 TOP -> 동일 반경 BOTTOM 순서대로 간다.

---

**플래터가 2개일 때**  
더 작은 반경으로 가기 전에 다음 플래터로 이동  
1번 PLATTER TOP 섹터 -> 1번 PLATTER BOTTOM 섹터 - > 2번 PLATTER TOP 섹터 - > 2번 PLATTER BOTTOM 섹터

---

**디스크 접근 시간**  
Disk access time = positioning time + transfer time  
원하는 섹터를 읽기 위해 arm을 이동하는 시간 = positioning time  
데이터가 DRAM으로 이동하는 시간 = transfer time  

걸리는 시간 : 트랜스펄 타임 << 포지셔닝 타임

Positioning Time = Seek Time + Rotational Time

Seek Time  
track 간에 arm을 이동하는 동작을 seek이라고 하고 이때 걸리는 시간을 seek time이라고 함.

Rotational Time  
읽고자 하는 sector가 회전할 때 arm에 도달하는 시간 rotational time.

플래터의 홈이 작으면 작을수록 데이터를 많이 저장할 수 있음. 홈이 작을수록 바늘을 놓는 작업이 미세하게 조정되어야하여 시간이 많이 걸린다. -> Seek Time이 크다.
기술이 발전할수록 RPM이 높아져서 Rotational Time은 줄어든다.

따라서, Seek Time >> Rotational Time

---

**디스크 드라이버**  
디스크 섹터 혹은 블럭  
파일 시스템에서 디스크에 섹터를 넣으려고 할 때 섹터 넘버를 던져줌.  
디스크는 디스크 내부에서 원하는 섹터로 들어갈 수 있게함.  

디스크 컨트롤러  
디스크 컨트롤러는 Mother보드에 기본적으로 존재함.(디스크 내부에 있을 수도 있음)
CPU랑 디스크 사이에서 둘이 통신할 수 있도록 해줌.  
EX) CPU가 디스크섹터 읽고 싶은 것 디스크 컨트롤러한테 전달하면 디스크에서 읽어서 다시 CPU에 전달해줌.  

---

**디스크 캐시**  
CPU에서 디스크 컨트롤러한테 디스크 특정 섹터를 읽어달라고 요청을 하게 되면,
1. 디스크 컨트롤러는 해당하는 섹터를 찾아
2. DISK DRAM에 먼저 저장함.
3. DISK DRAM에서 DRAM에다가 저장함.
4. CPU는 DRAM에 올라와있는 섹터 정보에 접근함.

---

**디스크 스케줄링**  
디스크 접근 시간을 줄이는 기준 : seek time과 rotational time을 줄이는 것.
seek time이 현저히 크므로 seek time을 줄이는데 초점을 맞추고 있다.

FCFS(FIFO방식) : FIRST COME, FIRST SERVICE 가장 안좋은 방법!!
큐로 먼저 들어온 request를 먼저 서비스 해줌.

SSTF (Shortest Seek Time First) 큐에 있는 것 중 현재 위치에서 제일 가까운 순서대로 찾아가는 것. 이 방법은 Starvation 문제가 발생.

SCAN 스칸 (엘리베이터 알고리즘) : 한쪽 방향으로 쭉 서비스를 실행하면서 가고 중간 중간에 해당하는 실린더 번호를 서비스해줌. 끝 부분에 부딪히면 방향을 바꿈.

C-SCAN(SCAN과 비슷함) : 한쪽 방향으로 쭉 가는 건 동일하나 벽에 부딪히면 반대방향으로 방향을 바꾸지 않고, 반대방향 끝쪽으로 이동하여 다시 서비스를 재개함.

LOOK : Request Queue 안에 있는 가장 큰 값과 가장 작은 값까지만 이동하는 함. 가장 큰 값이나 가장 작은 값에 도달하면 방향을 반대 방향 처음 부분으로 돌아가는 것이 아닌 방향만 바꿔서 다시 서비스를 재개함.

CLOOK : C-SCAN과 차이점은 끝까지 가지 않고 Request queue에 있는 제일 큰 값까지만 가고 반대 방향 끝으로 돌아오는데 돌아올 때도 Request queue에 있는 가장 작은 값으로 돌아온다. 이렇게 되면 끝까지 가게 되는 불필요한 움직임을 줄임으로써 Seek time을 줄일 수 있다.

LOOK과 C-LOOK 방법을 실제 디스크에서 많이 사용함.

---

**디스크 포멧팅**  
Disk formatting : 디스크 사용하기 앞서 포매팅을 해야함.

디스크 low-level 포매팅 : 디스크를 섹터단위로 나누는 것, 디스크를 특정한 숫자로 채움

Logical formatting : 파일 시스템 초기화하는 과정, 파일 시스템이 가지고 있는 데이터 구조를 완전히 리셋시켜버림. 빠른 시스템 포매팅

low-level formatting + logical formatting : 디스크의 모든 부분을 0으로 꽉 채워버림. 그리고 logical formatting을 실행함. Full file system formatting이라고 부르며 느린 포맷임.

---

**File system**  
File system에서는 disk에게 block 단위로 데이터를 요청하게 됨.  

Block = 2^n * sector  
만약 n이 2면 => 2 sectors = 1 block 블록 단위로 섹터를 묶음.  
ex) 보통 1 block = 4096 = 8 * sector / 1 block = 16k = 32* secotr  

File identity : 커널에서는 디스크에 있는 파일을 구별하기 위해 유니크한 아이디를 사용함. 반면 유저들은 파일 이름을 가지고 구분함. inode

File descriptor : disk엔 file_date와 file_info를 저장함 open함수를 호출하면 DRAM에 INFO를 올리게 되고,  file_address 정보 File descriptor로 반환함.

inode : file info를 담고 있는 파일 블록

file descriptor table : fd를 관리하는 테이블. 프로세스당 하나만 가지고 있음.

커널 내에는 file table이라는 것이 존재. fd와 file info 사이에 존재하는 매개 오브젝트 file offset, reference count, pointer 등이 저장되어있음.


fd table / file table / v-node table 순으로
(v-node는 메모리상에 존재하는 inode를 뜻함 = in=memory inode)  
open 함수 실행되게 되면  
1. disk에 a.txt에 대한 inode를 메모리 상으로 호출
2. file table에 빈 entry를 하나 찾아서 v-node를 포인터 하도록 시킴.
3. fd table 가서 빈 entry를 하나 찾아서 file table을 포인터 하도록 시킴.

프로세스를 생성하면 fd table이 자동으로 하나 만들어지는 0번에는 stdin 1번에는 stdout 2번에는 stderr를 할당함. 처음 file open을 한다면 무조건 fd는 3번부터 사용하게 됨.  

**새로운 abc라는 파일을 생성한다면**  
1. 중복된 이름의 dir이 있는지 검사함.
2. 없으면 빈 엔트리에 새로운 디렉토리 엔트리를 생성함.

**abc 파일을 삭제하고 싶다면**  
1. abc라는 파일을 찾아감.
2. 해당 파일이 들어있는 엔트리를 invalid 상태로 바꿈. 비어있다는 뜻임.
3. 결론, 파일을 삭제한다고해서 파일을 삭제하고 데이터를 삭제하는 것이 아니라 단순히 디렉토리 엔트리만 invalid 처리함.
파일 삭제하는 함수가 unlink() 함수, remove(), rmdir()

---

**Logical Block and Pyhsical Block**
파일시스템에서는 디스크 단위를 블록 단위로 봄  
파일이 연속적으로 보이는 뷰가 Logical View  
파일에서 연속적으로 보이는 부분을 block 단위로 나눴을 때 Logical block  
디스크에 나뉘어져서 담겨있는 형태의 뷰 Physical View  
디스크에 있는 각각의 블록을 Pyhsical Block / 번호를 physical block number  
Logical block과 Physical block에 대한 관계 mapping  

Allocation Methods / 할당 방법
- Contiguous allocation : 한 파일을 구성하고 있는 블록들을 디스크에 연속적으로 배치

문제점 : External Fragmentation(외부적 단편화), Unpredictability for the size of the file(파일 크기 예측 불가)

- Linked allocation : 디스크 블록을 링크드리스트 형태로 관리함. 블록을 흩뿌림
문제점1 : Large searching overhead(random file access) 접근하기가 힘듦. 링크드리스트 특징. 중간에 있는 값에 접근하려면 모든 노드를 순차적으로 방문해야함.

문제점2 : 다음 공간을 나타내는 주소를 저장하는 공간이 4byte 할당되기 때문에 공간이 부족할 수 있다. 하지만 현대 시대에서는 해당되지 않은 이야기이다.

Linked Allocation – FAT에서 Cluster를 사용중이다.

File Allocation Table(FAT) : FAT 방식은 포인터 정보(4Byte)만 따로 빼서 테이블에서 관리함.

FAT 테이블에서 각각 엔트리의 사이즈가 16이면 FAT16, 32면 FAT32라고 부른다.

- Indexed allocation : 인덱스 블록에 physical block 주소를 저장함. logical block index에 physical block 주소를 저장함.

Multilevel index : 1st index 블록은 -> 2nd 블록 가르킴. 2nd 블록은 physical 블록을 포인터함.

Combined scheme : 실제로 사용되어지는 방법.
- Direct block pointer
- Single Indirect block pointer
- Double Indirect block pointer
- Triple Indirect block pointer

---
**Free Space Management**

Free block list : 빈 블록을 관리하는 리스트

새로운 파일을 만들어서 block을 사용하려고 하면 Free block list에서 빈 블록을 찾아서 할당 해줌.

한 파일에서 블록을 삭제하면 삭제된 정보를 Free block list에 저장하게 됨.

Bit map : Free block list를 관리하는 방법

Bit map 인덱스 0번은 physical block 0번을 의미, 1번은 physical block 1번을 의미. 비트 내부 값이 0이면 Free 상태 / 내부 값이 1이면 allocated 상태

**Buffer Cache**

블록을 읽을 때 동작 - Read  
App에서 Read를 써서 디스크를 읽으려고 함. File System은 디스크의 어떤 블록인지 알려줌. B0블럭을 읽으려고 한다. 1. 만약 최근에 읽은 적이 있어서 Buffer Cache에 존재한다면 Cache hit이다. 2. 읽은 적이 없어서 버퍼 캐쉬에 존재하지 않고 디스크에서 읽어와야하는 상황이라면 Cache miss이다.

블록을 쓸 때 동작 – Write  
App에서 Write하려고 하면, File System은 B0에 있다는 것을 확인하고 캐쉬에 찾아감.
1. 캐쉬에 존재하지 않으면 cache miss가 발생하고 B0를 Buffer Cache에 저장하고 바로 리턴함. 리턴했다는 것은 write함수를 완료했다는 의미. Write할 때 디스크에 접근하지 않으므로써 성능을 높임. 2. 다시 B0를 Write하려고 그러면 Buffer Cache에 존재하기 때문에 Cache hit이 발생함. Buffer Cache에 새로운 Write 내용을 업데이트 하고, 바로 리턴함. 이때도 디스크에 데이터를 저장하지 않음.

중간에 전원이 나가버리면 앱에서 Write했던 내용들이 손실이 나버림.
이 문제를 해결하기 위해 Buffer Flushing daemon(kernel process) 커널 내에서 이 프로세스가 주기별로 돌아감 10초~20초 사이로

---
**Flash Memory**

특정 단위로 데이터 접근해야하는 디바이스 -> Block Device Ex) Disk
바이트 단위로 데이터 접근이 가능한 디바이스 -> Character Device Ex) Keyboard, flash-memory

Flash Memory는 하나의 구간을 Page라고 함. (Disk에 Sector와 상응하는 말임)
Page의 읽고 쓰고 히는 기본 단위는 512바이트

특정 개수만큼 영역을 나뉜 단위를 Erase Unit이라고 함(Block이라고 표현하지만 다른 개념임). 보통 단위가 128k(256page)임

Erase-before-write
플래쉬 메모리의 특정 page에 데이터를 저장하고 그 공간에 다시 Write를 하려면, Erase Unit 전체를 깨끗하게 만들고 난 뒤에 OverWrite를 할 수 있다.

**FTL(Flash Translation Layer)**
Flash Memory 안에 저장되어 있는 테이블이다.
기존의 파일시스템과 다르게 플레쉬 메모리는 READ/WRITE/ERASE가 존재할 뿐만 아니라 읽는 단위도 Sector가 아닌 Page 단위이기 때문에 호환이 안됨. 그렇기 때문에 나온 것이 FTL이라는 테이블이다. FTL이 플레쉬 메모리에 펌웨어로 들어가있다. 

**SSD – Solid State Device 반도체 저장 장치**
장점  
읽는 속도가 엄청나게 빠르다. 순차적 데이터 Write 속도가 아주 빠르다.
GB당 데이터의 비용이 작다. 파워 소비도 작다. 플래터 같은 기계적인 움직임이 없다

단점  
블록 단위로 저장할 수 있는 횟수가 10만번으로 제한됨. 10만번 넘으면 더 이상 사용할 수 없음.
랜덤 Write가 느림. Erase가 많이 발생하면 성능이 저하될 수 있다.

---

**Process Concept**

프로그램 : 컴퓨터에서 컴파일 해서 나온 실행파일, 디스크에 저장된 수동적인 엔티티

프로세스 : 실행되는 프로그램, 엑티브 엔티티, 자기만의 리소스와 프로그램 카운트를 가지게 됨. 자신만의 메모리 공간과 컨트롤 정보를 갖게 됨.

실행 파일의 메모리 구조 : 맨 밑부터 text, data, heap, stack

**Process State**
new / running / waiting / ready / terminated

pid = fork() 하면 프로세스가 하나 만들어짐  
new : 프로세스가 초기화되는 단계  
ready : 해당 프로세스를 CPU에 올리기만 하면 바로 돌아갈 수 있는 상태를 뜻함  
running : 프로세스를 CPU 상에 올려놓고 실행중인 상태  
waiting : sleep() 함수를 사용하면 waiting 상태로 전환됨. sleep(10)이면 10초가 지난 다음 ready 상태의 새로운 프로세스를 running 상태로 만듦, read() 함수를 사용하면 disk 값을 읽어들이는데 시간이 많이 걸리기 때문에 cpu에서 waiting 상태로 두고, 새로운 프로세스를 가져와서 running 상태로 만듦

---

**Process Control Block (PCB) - 프로세스의 정보를 담아두고 있는 구조체**

- Process State : read, running, waiting, halted 등등
- Program Counter : 그 다음 실행할 명령어의 위치를 가르키고 있음
- Cpu Register
- Cpu 스케쥴링 정보
- 메모리 매니지먼트 정보
- CPU를 얼마나 사용하고 있는지
- I/O 상태 정보

---

**프로세스 스케쥴링**
1. 멀티 프로그래밍 : 여러 프로세스를 동시에 실행하는 동작, CPU 활용성 최대
2. 타임 쉐어링 : 여러 사용자가 컴퓨터를 동시에 사용하게 해주는 동작, 각 사용자가 해당하는 컴퓨터를 자신만의 컴퓨터인 것처럼 느끼게 해주는 것.
3. 프로세스 스케쥴러 : 멀티프로그래밍 + 타임쉐어링을 지원해주는 것이 프로세스 스케쥴러

---

**Context Switch**

Context Switch 하나의 프로세스가 CPU를 사용 중인 상태에서 다른 프로세스가 CPU를 사용하도록 하기 위해, 이전의 프로세스의 상태(문맥)를 보관하고 새로운 프로세스의 상태를 적재하는 작업을 말한다.
Process Switch == Context Switch
Context : CPU 레지스터의 값의 묶음과 프로세스 상태를 의미함.

Context Switch 예제 : CPU 레지스터에 Context값을 교환하는 것.
CPU에서 P0가 실행중이라면 P0 레지스터에 Context 값 Load해서 실행하다가
P1으로 바뀌어야한다면, 현재 CPU의 P0의 Context 값을 P0 레지스터에 저장함.
CPU에서 P1의 레지스터안 Context 값을 로드해서 CPU 레지스터에 저장하고 실행

---

**CPU Burst / I/O Burst Cycle**

CPU는 두 개의 사이클로 구성되는데
- CPU 실행 사이클인 CPU burst
- I/O Wait 사이클인 I/O burst

---

**프로세스 스케줄러**

레디큐에서 ready 상태로 대기중인 프로세스 중 하나를 선택하는 것이 프로세스 스케줄러의 역할

디스페쳐 : 선택한 프로세스를 CPU에 올려서 실행하는 역할

언제 CPU로 프로세스를 선택해서 올릴까?
1. running 상태에서 waiting 상태로 갈 때
2. running 상태에 있는 프로세스의 time slice가 모두 끝나면 ready 상태로 보내고, 다른 프로세를 선택해서 running 상태로 보냄
3. 프로세스가 실행되다가 갑자기 죽게되면 ready 상태에 있는 프로세스 중 하나를 올림

프로세스 스케줄러 종류 2개
1. Non-preemptive(비선점) : 자발적으로 놓지 않는 이상 계속 프로세스를 실행하게 해줌(종료한다던지 I/O한다던지 이런 경우가 아니라면 계속 프로세스 실행하게 해줌)
2. Preemptive(선점) : 실행되고 있는 프로세스를 강제적으로 중지시키게 해줌. 필요한 프로세스가 있다면 현재 프로세스 ready 큐로 보내고, 필요한 프로세스 가져옴.

Dispatcher : 프로세스가 선택한 프로세스를 CPU로 올리는 역할을 함.

1. ready queue로 context switching 해줌
2. 프로그램 카운트에 예전에 정지했던 부분을 넣어주어 그 부분부터 자연스레 실행되게 함.
3. 프로세스를 유저모드로 올려야함. 일반적으로 프로세스는 유저모드에서 돌아감. ready queue에 있는 것들은 kernel 모드이기 때문에, 실행될 때 유저모드로 바꿔줘야함.

Dispatch latency(디스페치 지연속도) : 위 3가지 동작을 할 때 걸리는 시간. 요즘 시대에는 컴퓨터 성능이 좋아서 디스페치 라텐시가 무시됨.

---

Scheduling Algorithms 스케줄링 알고리즘

1. First-Come, First-Served : non-preemptive(비선점) : 자발적으로 놓지 않는 이상 계속 실행
2. Short-Job-First : (어떻게 보면 priority 스케줄링임. 버스트 타임이 짧은 것이 기준) CPU 버스트 타임이 가장 적은 것들부터 우선적으로 실행
3. Priority Scheduling : 각 프로세스에게 우선 순위 정수가 할당되게 됨.
이때 스케줄러는 priority가 가장 높은 순서대로 CPU에 프로세스를 할당함. (작은 값이 우선순위가 되는 경우도 있는데 OS에 따라 상이함)
4. Round-Robin Scheduling : 대부분의 OS에서 이 방법을 사용함. 이 방법을 사용해야 사용자는 여러개의 프로세스가 동시에 돌아가는 것처럼 느낌. 각 프로세스가 마치 CPU를 독점하는 것처럼 느끼게 해줌.
5. Multilevel Queue : Ready queue를 여러개를 만드는 방법. 예를들어 3개의 Ready Queue가 있는데, A는 FIFO, B는 Round Robin, C는 Priority Queue
Example : Priority-Based multilevel queue Scheduling
6. Multilevel Feedback-Queue : Starvation이 발생할 수 있는 상황을 방지한 것이 Multilevel-Feedback-Queue 오랫동안 사용하지 않은 프로세스는 우선순위를 올려줌. CPU에서 많이 사용된 프로세스는 우선 순위를 밑으로 내림.

---

**쓰레드의 등장**

프로세스의 문제
1. 프로세스 생성하는데 시간이 많이 걸림
2. 컨텍스 스위칭 오버헤드가 크다.
3. 동기화 오버헤드가 크다. (lock, semaphore)

쓰레드는 자신만의 context를 가진다 ex) Thread ID, 프로그램 카운터, 레지스터 셋, 스택 많은 OS들이 멀티쓰레드 프로그래밍을 지원한다.

CPU의 실행단위는 프로세스가 아니고 쓰레드이다. 스케줄링 Unit 단위도 쓰레드가 됨.

---

**Procedure Calls vs Multithreading**

공통점  
1. 다른 함수 혹은 쓰레드에서 사용된 지역변수에는 접근이 불가능하다.
2. 전역변수와 힙 영역은 다른 함수 혹은 쓰레드들과 공유가 가능하다.

차이점  
1. 쓰레드는 쓰레드당 하나의 스택을 가진다. 쓰레드가 실행되는 도중에는 각자 자신만의 스택을 가진다.
2. Procedure Call은 함수 하나하나 차례차례 실행이 된다. 하지만 쓰레드는 2개 동시에 돌아감.

---

**Benefits 쓰레드의 장점**

1. 첫 번째로 중요한 것은 쓰레드가 사용하는 메모리 공간은 작다. stack과 TCB 공간만 있으면 된다. 반면 프로세스는 엄청난 메모리 공간을 잡아먹음
2. 쓰레드 생성 시간이 짧다. 작은 메모리 공간을 할당하기 때문.
3. Context Switching Overhead가 작다.
4. 쓰레드간의 동기화 부하가 작다.

---

**Memory Management Strategies**

Memory Management for Multiprogramming

Dram 상에 여러 가지 프로세스 올라가 있고, 여러 개의 프로세스가 올라가 있음.
Protection : 다른 프로세스가 할당된 메모리를 침범하지 못하도록 막는 것

다른 프로세스가 내 프로세스의 메모리를 침범해서 디버깅을 해야하는 상황이 발생할 수 있다. 또한 프로세스가 OS가 돌아가는 메모리를 침범하면 컴퓨터가 다운되는 현상이 발생할 수 있다.

**Protection 방법**
Base Register(현재 레지스터의 가장 낮은 번지) + Limit Register(프로세스 메모리의 크기를 저장) 2개를 설정해주면 다른 프로세스 메모리 침범하지 않음.

---

**Contiguous Memory Allocation**

문제점 : Process 1이 이렇게 분할되어 Virtual 공간 상에 저장되어 있으면 17000이란 값을 넣었을 때 문제 없이 통과되어버리는데 실제론 Process2의 공간이라 메모리를 침범한 것이 되어버림.

Solution프로세스 메모리를 할당할 때는 항상 연속적으로 DRAM상에 할당해야함.

외부적 단편화(메모리 조각화)가 발생할 수 있음.

공간이 있음에도 불구하고 들어갈 수가 없음.

해결하는 방법
1. Compation ; 가비지 콜렉션과 동일한 말임. 가비지 콜렉션 실행시 공간을 앞당기는데 카피 동작이 많이 일어나고 성능 저하에 영향을 미침. 그래서 이 방법은 사용할 수 없음
2. Non-Contiguous Address : 그냥 빈 공간에 저장하자. 이 방법이 paging

---

**Paging**

Paging : 페이징은 프로세스의 메모리 공간을 DRAM 상에 흩뿌려서 관리하겠다.

Dram을 페이지 단위로 쪼갬, 파일 시스템의 block과 Dram의 Page은 서로 유사한 관계임

프로세스 A도 page 크기로 나눔. 첫 번째 페이지부터 쭉 0번 1번 2번 순서대로 번호를 매김. DRAM에서 page 단위로 쪼갠 후 그 단위를 Frame이라고 부름.
Contigous 하게 저장하지 않고 페이지 단위로 흩어져서 저장하게 됨.

logical address를 반으로 짤라서 앞쪽은 page number(p) 뒤쪽은 page offset(d)라고 함.
page number는 page table을 인덱스 하기 위해서 사용
page offset은 page가 바이트 단위로 되어 있다면, 어드레스가 몇 번째에 존재하는지 나타내주는 것이 page offset이다.

1. p 인덱스를 참조하여 f가 저장되어있음을 알고 f를 반환
2. 앞쪽은 f 뒤쪽은 d를 붙여만듦.
3. 해당하는 특정한 위치에 physical memory를 가리키게 됨.

m은 V.A.S가 2^4이기 때문에 m = 4  
n는 page 사이즈가 2^2이기 때문에 n = 2  
page offset = q는 n과 같기 때문에 q = 2  
page number = p는 m – n이기 때문에 p = 2  

Virtual Address 0번에 해당하는 Pyhsical Address 구하려면  
page number : 0 / pagesize == 0 / 2^n = 0  
page offset : 0 % pagesize == 0 % 2^n = 0  
구하는 법 >> page table 0번에 가서 매핑되는 frame 번호를 구함. 해당 5번 frame에 가서 offset이 0이기 때문에 0번째에 해당하는 값이 대응되는 physical Address 값임.

Virtual Address 13번에 해당하는 Pyhsical Address 구하려면  
page number : 13 / pagesize == 13 / 2^n = 3  
page offset : 13 % pagesize == 13 % 2^n = 1  
구하는 법 >> page table 3번에 가서 매핑되는 frame 번호를 구함. 해당 2번 frame에 가서 offset이 1이기 때문에 1번째에 해당하는 값이 대응되는 physical Address 값임.  

그러나 이런 방법은 사람이 생각할 때 개념이고, 실제 연산 동작은 다음과 같음.
13을 2진수로 나타내면 1 1 0 1임. 이것을 p bits, q bits 만큼 나누면 1 1 / 0 1로 나뉘게 되고 이 값이 3과 1임. Physical Address는 3에 해당되는 값 2를 다시 이진수로 나타내면  
1 0임. 해당 Frame과 1 0과 offset 0 1을 붙여 만든 1 0 0 1이 되고 이 값은 9임.
따라서 전체 physical bytes 단위 중 9번을 보면 됨.  

---

**Internal Fragmentation 내부적 단편화**

한 페이지 크기가 4096일 때, 512만큼만 할당됐다고 가정함. 그럼 나머지 사용하지 않는 공간을 보고 internal fragmentation이라고 함.

Pagesize가 증가할수록 내부적 단편화가 증가함. pagesize가 감소할수록 내부적 단편화가 감소함.

내부적 단편화를 해결하기 위해, 페이지 사이즈를 줄이면 줄일수록 페이지 테이블 엔트리 개수가 증가하는 현상이 발생.

---

**Memory Protection**

메모리에서 Shared lib + text(code) 영역은 절대 건들면 안됨 write불가, read만 가능

**Valid-Invalid Bit**

V.A.S의 일부분의 페이지는 DRAM에 매핑(저장)되어 있다. 그런데 일부분의 페이지는 DRAM상에 매핑되어있지 않을 수 있다.

---

**Shared Pages**

시스템에 40개의 텍스트에디터 프로세스가 있고, 150kb코드공간와 50kb의 데이터 공간 사용하고 있다고 가정 그렇다면 전체 메모리 사용량은 40 *200kb = 8000kb
메모리 상에 올라간 text code는 Read-only여야함.
1. 보안상 문제
2. 2.오작동 막기 위해서

---

**Segmentation**

Segmentation : 페이지의 크기로 나누는 방식과 달리 text code, data, stack 단위로 메모리 크기를 나눈다.
Symbol table : 함수이름, 전역변수 이름을 관리하는 테이블

Segment : 프로그래머가 인지할 수 있는 논리적 엔티티, 프로세스을 구성하는 있는 요소의 특성 별로 나누어진 엔트리.
세그먼트의 종류
1. Text Code 2. Global Variables 3. Heap 4. Stacks 5. Standard C library

세그멘테이션(Segmentation fault의 segmentation 과 다름.)

---

**Case Study : Intel Pentium - 실제 인텔 CPU에서**

**Segmentation Unit 동작 원리**

1. s를 통해 해당하는 세그먼트 테이블 엔트리에 찾아가서, limit 검사 후 base값을 구함.
2. gdt인지 ldt인지 판단함. 그리고 base + offset를 한 32-bit linear address를 반환

**Paging Unit 동작 원리**

인텔에서는 Outer Page table을 보고 Page Directory 라고 함.
p1 인덱스를 참조하면 그 안에 DRAM상에 존재하는 Page table의 page의 위치가 나옴.
그 위치에서 p2 인덱스 값을 참조하면 Frame 번호가 나옴. 해당 메모리를 접근하고 d값을 더해주면 됨.

프로세스1, 프로세스2가 있다고 가정하면 P1을 쓸 때는 CR3레지스터에 P1의 Page Directory 값을 저장하고, P2를 쓰게 되면 Context Switiching 될 때 CR3레지스터가 P2 Page Directory 값으로 바뀌게 된다.

---

**TLB(Translation Look-aside Buffer)**

TLB는 캐쉬로 구성되어 있다.

인텔 프로세서 CPU, MMU가 존재함.
CPU에서 V.A(p, d)를 MMU에 전달함. MMU는 p를 뽑아서 page-table에 접근함.
p인덱스에 f값이 저장되어있다고 가정. (f, d)를 합쳐서 메모리를 접근하게 됨.

cpu는 접근속도가 빠르지만 메모리는 접근 속도가 낮다. CPU가 매번 메모리를 접근할 때마다. 매번 메모리에 페이지 테이블을 접근해야 함. 계속 페이지 테이블을 접근하게 되면 
V.A를 P.A로 변환할 때 성능저하가 생김. 그래서 나온 것이 TLB임.

---

**Virtual Memory Management**

Demand paging 디멘드 페이징
Swapping 스와핑
Page replacement 페이지 리플레이스먼트
Memory mapped file

---

**Demand Paging**

왜 전체 프로그램을 메모리 상에 올려야할까?  
필요할 때마다 메모리로 로드하게 되면 그만큼 메모리 공간을 최소화시킬 수 있다.
이것이 Demand Paging 기법이다.

---

Page Fault (아주 중요한 용어)

위 그림에서 3번 페이지에 접근하려고 함.  
페이지 테이블에서 3번 인덱스에는 I로 체크되어있음(메모리에 없음)  
이 현상을 보고 Page Fault라고 함.  

페이지를 접근하려는데 메모리에 존재하지 않아서 실패한 현상을 Page Fault라고 함.  
invalid bit가 I로 된 페이지를 접근할 때 Page Fault가 발생한다.  

Page Fault Exception이 발생하여 다음과 같은 동작이 수행됨.

Page 2번을 접근하려고 그런다. 그런데 valid/invalid bit가 I라서 메모리상에 존재하지 않는다. 이때 CPU는 Page fault exception을 내부적으로 발생시킨다.(이벤트임)

그러면 Page Fault handler(C코드로 되어 있음)가 실행된다.

그러면 Virtual Memory SubSystem에 구축된 어떤 함수를 호출

함수 내에서 무슨 동작을 하냐면  
1. 페이지 폴트 원인을 검사
2. 원인에 따라서 (1)디스크 코드/데이터 로드 (2)프로세스 강제종료 위 둘 중 하나 실행

---

**Effective Access Time**

(1) 페이지 폴트 exception을 발생시키고, 페이지폴트 핸들러 발생까지 걸린 시간은
1msec보다 적다. CPU 명령어만 실행하면 되기 때문에 시간이 엄청 적게 걸린다.

(2) Disk latency 디스크를 읽는 시간은 8msec 가정(로테이셔널 + 시크 + 데이터트랜스펄)

(3) 페이지 테이블 설정하고 정지된 프로세스 재시작하는 시간 : 1msec, CPU 명령어라서

(1) + (2) + (3) 모두 합치면 8msec 정도 걸리는데 Disk 어세스가 대부분을 잡아먹음

페이지 폴트가 많이 발생할수록, 코드와 데이터를 읽어와야 하기 때문에 디스크I/O가 증가함. 성능이 떨어질 수 있다.

Effective memory access time 계산  
ma = 200 nanosec / page fault rate 1/1000  
(1-p)*ma + p * page fault time  
(1-1/1000)*200 + 1/1000*8,000,000  
= 199.8 + 8000 = 8,1998 = 8,2 used  

---

**Page Replacement 페이지 교체**

그림과 같이 DRAM에는 모든 프레임이 페이지 별로 저장되어있음.
load M은 DRAM상에 있기 때문에 page fault가 발생하지 않는다.
Load M을 실행하면  M페이지에 접근한다고 가정, M은 Page 3번에 있음.
Page Table 3번 인덱스에 DRAM에 존재하지 않음. invalid/valid bit가 I임.
그렇기 때문에 Page Fault가 발생하고 디스크에서 read해와야하는데 DRAM에 공간이 없음
이때 어떻게 할 것인가?

**Sol 1) Process Termination**

페이지 폴트를 야기한 프로세스를 종료해 줌. 원시적인 방법
V.A를 사용하지 않은 Realtime OS 등은 쓰레드와 테스크를 강제적으로 종료시킴.

V.A가 존재하는 OS에선 다음과 같은 방법(Sol2, Sol3)을 사용할 수 있음.

**Sol 2) Process Swapping**

만약 프로세스 1번이 실행되는데, 공간이 없다면 프로세스 2번이 사용 중인 DRAM의 공간을 모두 디스크로 저장시켜버림.

**Sol 3) Page Replacement**
페이지 교체

현재 DRAM에 빈 프레임이 하나도 없음. 101번 page에서 page fault가 발생
디스크로 쫓아낼 victim frame을 하나 선택하고 디스크로 back-up시킴.
이것을 보고 page out이라고 부름. (swap out은 프로세스 모든 프레임을 저장시키는 말)

1) 100번 페이지를 0으로 바꾸고 I 상태로 바꿈
2) page in을 하여 101번째에 해당하는 frame을 DRAM으로 가져옴.
3) 101번 페이지에 해당하는 frame 인덱스 f를 page table에서 설정해주고 v로 바꿈

Process Swap보다 효율성이 좋다. 필요한 페이지만 하나씩 가져오면 되기 때문에.

---

**Page Replacement : Modify (Dirty) Bit**

Victim이 Page 2번인데 Read-only라고 가정. 그렇다면 한번도 변경되지 않은 것임.
그렇기 때문에 Page out을 할 필요가 없음. 디스크에 저장 없이 그냥 날려버림.

Page 3은 그냥 업데이트 된 상태 (Dirty 상태) 근데 Page 3번이 Victim이라고 가정,
그러면 최신 데이터이기 때문에 무조건 저장해야함. page out이 실행
이런 동작을 이룰 수 있도록 Modify Bit라는 것을 추가함.
만약 CPU가 어떤 페이지를 업데이트하면 MMU에서 WRITE 했다고 판단하고 Modify Bit를 1로 셋팅함.

이렇게 Modify Bit가 1인 상태에서 Victim Page로 설정됐다면, page-out 함.
Modify Bit가 0이면 그냥 날린다. 이 과정으로 디스크 I/O를 최소한 줄이겠다.

---

페이지 교체 알고리즘 4가지

1. FIFO Page Replacement : 메모리 상에서 가장 오랫동안 머물렀던 페이지를 Victim으로 선택해서 교체를 하겠다.
2. Optimal Page Replacement : 미래에 가장 오랫동안 사용되지 않을 것 같은 페이지를 교체하는 것임.
3. LRU Page Replacement : 먼 미래를 예측하기 위해서 가까운 과거를 살펴보겠다.
가까운 과거에 접근된 페이지는 가까운 미래에 접근될 가능성이 높다. 먼 과거에 접근된 것을 페이지 교체한다.
4. LRU-Approximation Page Replacement : Second-Chance Algorithm이 사용됨.

---

**Thrashing**

짧은 시간 동안 Page Fault가 엄청 많이 일어나는 현상을 쓰레싱이라고 한다.

쓰레싱의 주원인이 무엇이면 메모리 크기가 작다는 것임. 많은 프로세스를 동시에 실행시키면 메모리가 작다는 것. 쓰레싱을 해결하려면 메모리 크기를 증가시키는 방법 뿐이다.

---

**Locality Model(지역성)**

함께 활발히 접근되는 페이지의 집합 (반복적으로 접근되는 Value, 메모리 위치, 파일 블록, 관련된 저장 위치)

Page 11 -> 12 > 10 접근  
서로 인접한 Page를 접근하는 현상을 보고 Spatial Locality 공간적 지역성.

Temporal Locality : 동일한 변수, 메모리 위치, 파일 블록이 반복적으로 접근될 때 변수 또는 메모리 위치들이 Temporal Locality를 가졌다고 함. int I 등

Spatial Locality : 서로 인접한 것들이 항상 동일하게 함께 접근되는 현상이 있으면

---

**Working-Set Model**
가장 최근에 데이터 안에 접근된 페이지의 총 개수를 워킹셋이라고 함. 시간에 따라서 바뀜.

델타가 너무 작으면 프로세스가 활발히 사용하는 메모리의 워킹셋을 예측하기가 쉽지 않음.

델타를 너무 크게 해버리면 자주 접근하는 페이지 이외에도 자주 사용하지 않는 페이지도 워킹셋에 포함되어버리면서 불필요하게 메모리를 많이 잡아먹음.

전체 시스템에서의 워킹셋의 합이 D
D = 워킹셋의 총합.

만약 D > m (DRAM 메모리)이면 Trashing이 발생하게 됨.
D < m이면 Thrashing 발생안함.

---

**Synchronization 동기화**

쓰레드 또는 프로세스의 실행을 제어하는 동작을 보고 동기화라고 그럼.

프로세스 스케줄링이 쓰레드 스케줄링와 의미가 같다고 보면 됨.

멀티프로그래밍(여러개 프로세스 or 쓰레드 동시에 구동)을 하는 목적은 CPU utilization을 최대화하는 것이 목적. CPU가 놀지 않도록 해주는 것.

---

**Race Condition**
Counter++ 실제 어셈블리에서 동작

실행 순서에 따라서 특정한 시점에 서로 다른 값이 나왔다.

두 쓰레드가 공유 데이터를 접근하는데 병행적으로 동시에 접근할 때, 쓰레드의 실행 순서에 따라서 공유 데이터의 값이 달라지는 상황을 Race Condition이라고 함.

---

**Critical-Section(임계영역) Problem : 공유 데이터를 접근하는 그 프로그램의 부분 문제**

공유 데이터가 있는 부분이 크리티컬-섹션임

크리티컬 섹션에서 제어를 잘못해주면 Race Condition이 발생할 수도 있음.

Race-Condtion 문제를 Critical-section Problem이라고함.

이것을 해결해줄 수 있는 것이 Synchronization 동기화

---

**A Solution to Critical-Section Problem**

1. Mutual Exclusion 상호 배제
T0이 E.S로 들어가서 C.S로 들어간다. T1도 E.S로 들어가서 요청을한다.
한 쓰레드가 C.S를 접근 중이면 다른 쓰레드는 절대 C.S로 못 들어가게 한다.
T0로가 Exit.S로 빠져나와야 T1에서 접근이 가능하다.
한 쓰레드만이 크리티컬 섹션을 독점할 수 있게 한다
.
좀 더 완벽한 동기화를 위해서 2번이 필요하다.

2. Progress
T0가 먼저 C.S로 들어감. T1이 들어가려고 E.S에 들어갔는데 T0가 먼저 들어갔기 때문에 WAIT 중임. T0가 빠져나오면 완벽하게 빠져나오면 1초후에 들어간다던지 10초후에 들어간다던지 이러면 안되고 바로 들어가게끔 해줘야함. 이것을 Progress라고 함. T0가 빠져나온 순간에 바로 T1가 들어가도록! 

3. Bounded Waiting
T0가 C.S로 들어감 T1은 Entry로 들어가서 Waiting, T0가 빠져나왔는데, While문 통해서 T0는 다시 C.S접근 가능. 그런데 T1이 들어가는 것이 아닌 T0가 빠져나왔다가 바로 다시 들어가버리면 T1는 무한대로 대기만 하게 됨. Starvation 발생. 공평하게 T1도 들어갈 수 있게끔 허락해주는 동작을 보고 Bounded Waiting이라고 부름.

---

**Synchronization : Lock**

크리티컬 섹션, Race Condition 문제 동기화를 할 수 있는 가장 기본적인 매커니즘.
대부분 OS에서 제공하고 있음. Linux, Unix, Windows XP OSes.에서 mutex로 제공

acquire lock과 release lock이 있다. 각각 Entry 섹션, Exit 섹션에 대응됨.
다른 한 쓰레드가 lock을 잡고 있으면, 다른 쓰레드는 lock을 못잡고 Waiting

Mutex(Locks, Latches) Mutual Exclusion에서 MUT + EX, Locks이라고 많이 이야기함.

locked state : 쓰레드가 mutex를 hold 혹은 own 하고 있는 상태
unlocked state : 어떤 쓰레드도 mutex를 잡고 있지 않는 것.

---

**Semaphore : OS에서 제공하는 가장 기본적인 메커니즘.**

중요한 변수
Semaphore count == SeToken count 예제에서는 pCount로 나옴

중요한 함수
wait() : p함수라고 부름 / siganl() : v함수라고 부름
p는 세마포어를 만든나라가 네덜란드인데 Dutch(네달란드) Proberen-Test을 따서 P
v는 Verhogen – increment를 따서 v라고 함.
Wait 함수는 SemaCount를 1 감소시키고 / signal 함수는 SemeCount를 1 증가시킨다

---

**Deadlock Problem**

데드락이 발생할 수 있는 조건

1. Mutual Exclusion : 어떤 쓰레드가 Critical Section 사용중라면 다른 쓰레드가 들어갈 수 없다. 한 개의 쓰레드는 자원을 하나밖에 잡을 수 없다.

2. Hold and Wait : 한 쓰레드가 자원을 잡고 있으면서, 다른 쓰레드가 잡고 있는 자원을 요청하고 기다린다.
3. No preemption : 리소스를 잡고 있는 쓰레드만이 그 자원을 해제할 수 있음. 제3의 쓰레드나 OS가 해제할 수 없음.
   
4. (아주중요) Circular wait : p0, p1, ...p0이 있는데 p0은 자원을 갖고 있음, 그러면서 p1의 자원을 잡으려고 waiting 중이다. p1은 p2가 가지고 있는 자원을 잡으려고 waiting 중이다. 이런 식으로 쭉 pn-1도 p0가 선점 중인 자원을 가지려고 하고 있음.

---