# 🥇 정보처리 기사 실기 프로그래밍-구조체(structuer)

#정보처리기사 #실기 #프로그래밍 #정리 #구조체

---

# C언어로 구현된 프로그램을 분석하고, 실행결과를 쓰시오.

```c
#include <stdio.h> 
struct jsu { 
	char nae[12]; 
	int os, db, hab, hhab; 
}; 

int main() { 
	struct jsu st[3] = {
		 {"데이터1", 95, 88},
		 {"데이터2", 84, 91},
		 {"데이터3", 86, 75}
	};
	struct jsu* p; 
	p = &st[0]; 
	(p + 1)->hab = (p + 1)->os + (p + 2)->db; 
	(p + 1)->hhab = (p + 1)->hab + p->os + p->db; 
	printf("%d", (p + 1)->hab + (p + 1)->hhab); 
}
```

- **구조체는 다음과 같이 연속된 공간에 저장된 후 사용된다.**
![[Pasted image 20250312173438.png]]
문자열 저장시 끝을 의미하는 널문자('\0')가 추가되며,
영문, 숫자는 1Byte / **한글은 2Byte**를 차지한다.
![[Pasted image 20250312173740.png]]

1. `p = &st[0];` => 구조체 st의 첫번째 객체의 주소값을 저장
2. `(p + 1)->hab = (p + 1)->os + (p + 2)->db;`
	- `(p+1)->hab` : st첫번째 객체의 다음객체(`st[1]`)의 hab요소의 값
	- = `st[1]->hab = 84 + 75` = **159**
3. `(p + 1)->hhab = (p + 1)->hab + p->os + p->db;`
	- = 159 + 95 + 88 = **342**

### 답 : 159+342 = 501

---

>[!tip] 구조체 특징
> 구조체의 멤버를 지정할 때는 변수명 멤버이름 으로 지정하지만, **[변수명].[멤버이름]**
> **포인터 변수**를 이용해 구조체의 멤버를 지정할 때는 변수명 멤버이름 으로 지정한다 **[변수명]->[멤버이름]**

---

# 다음 c코드의 출력값은?
```c
struct Node {
    int value;
    struct Node* next;  
};

void func(struct Node* node) {
    while (node != NULL && node->next != NULL) {
        int t = node->value;
        node->value = node->next->value;
        node->next->value = t;
        node = node->next->next;
    }
}

int main() {
    struct Node n1 = {1, NULL};
    struct Node n2 = {2, NULL};
    struct Node n3 = {3, NULL};
    
    n1.next = &n3;  
    n3.next = &n2;

    func(&n1);

    struct Node* current = &n1;
    while (current != NULL) {
        printf("%d", current->value);  
        current = current->next;
    }
}
```

n1 - n3 - n2 순서
- 구조체의 주소를 메소드로 넘길경우 *값(value)* 은 반영이된다.

node->value = node->next->value;
- n1의 벨류는 n3의 벨류가 된다
node->next->value = t;
- n3의 벨류는 n1의 벨류가 된다.
node = node->next->next;
- 현재 노드를 n2로 바꾼다
- 이때, n2의 next는 없으므로 조건 종료

이때 n1부터 연결된 노드의 벨류를 프린트하면 3 1 2

### 답 : 312

----


