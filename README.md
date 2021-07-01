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
