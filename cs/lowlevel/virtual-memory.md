# Virtual Memory

## Physical vs Virtual Addressing

메모리는 word 하나를 저장할 수 있는 cell 여러 개의 묶음. 보통 word 크기는 1 byte임

각 cell은 고유한 physical address를 가짐. 단순하게, 이 physical address로 메모리를 참조해서 사용할 수 있음 (옛날 컴퓨터는 실제로 이렇게 돌아감)

반면 virtual addressing에서는, CPU가 virtual address를 사용해 메모리를 관리. 실제 메모리 접근은 physical address로 해야 하기 때문에, address translation을 거쳐서 메모리를 관리

## Virtual Pages

Virtual memory는 고정된 크기를 가진 virtual page 단위로 관리됨. DRAM을 일종의 cache로 볼 때, 각 page는 세 가지 상태를 가질 수 있음

- unallocated: 아직 할당되지 않은 상태
- cached: physical memory에 있는 상태
- uncached: physical memory에 없는 상태

DRAM cache miss가 나면 엄청 느린 disk를 써야 하기 때문에, DRAM cache는 writeback 방식으로 관리됨 (필요할 때만 disk write가 일어남)

### Page Table

앞서처럼 DRAM을 cache로 쓸려면 각 page의 상태를 알 필요가 있음. 어떤 페이지는 DRAM에 있고, 어떤 페이지는 disk에 있는지 알아야 함 :arrow_right: 여기에 page table을 사용

Page table은 PTE(page table entry)의 배열. PTE는 각 virtual page에 대응됨

PTE는 아키텍쳐마다 다르지만 physical page에 해당하는 주소와 valid bit가 있음. valid bit는 cached/uncached를 구분하는 데 사용

### Page Fault

DRAM cache miss를 page fault라고 부름

Page fault가 일어났을 때, 커널의 코드에서 이를 처리해야 함