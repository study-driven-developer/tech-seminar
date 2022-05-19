# Tech Seminar: String Matching Basics

## Rabin-Karp Algorithm

라빈 카프 알고리즘은 문자열 매칭 알고리즘이다. 

기본적으로 롤링 해시(Rolling Hash)를 사용한다. 

> **롤링 해시란?**
>
> $$
> H[i] = p^{M-1}S[i] + p^{M-2}S[i+1] + \cdots + pS[i+M-2] + S[i+M-1]
> $$
>
> 꼴로 만들어지는 해시를 말한다. 
>
> 예를 들어 "TEXT" 라는 문자열이 있다고 가정해보자.
>
> 그럼 이 문자열의 해시값은 다음과 같다. ( $p = 2$ )
>
> $$
> T * 2^3 + E * 2^2 + S * 2 + T = (84 * 8) + (69 * 4) + (83 * 2) + (84 * 1) = 1198
> $$

그럼 이제 실제 알고리즘의 동작과정을 살펴보자.

문자열 `"ABCDABCDAB"` 에서 `"BCD"` 를 찾는 문제가 있다고 해보자.

Brute-force 방법과 동작 과정은 거의 유사하다.

Pattern과 Text의 첫 3개 문자열을 비교한다.

![](https://velog.velcdn.com/images/dnr6054/post/71faf960-10dc-4cbc-90a2-cdb5d6fd74cb/image.png)

이 때, 직접 하나씩 비교하는 것이 아닌 Text에서의 첫 3개 문자열인 `"ABC"`에 대한 해시값과 패턴`"BCD"` 의 해시값을 비교한다. 

당연히 다르므로 다음 index로 넘어간다.

![](https://velog.velcdn.com/images/dnr6054/post/885893fe-8aca-47d7-83f3-7b34a7e15602/image.png)

이 때는 매칭에 성공하게 된다. 이런식으로 모든 문자열 조각과 패턴을 비교한다.

자 그러면 이런 의문이 들 수가 있다. 

> 아니 어차피 이렇게 하면 Brute-force랑 속도가 같은 거 아니야?
>
> 어차피 해시값 구하는 데 그 길이만큼 연산이 필요하잖아?

하지만, 앞서 말했듯 이 알고리즘에서는 롤링 해시를 사용한다.

$H[0] = A*4 + B*2 + C*1$ 의 값을 알고 있다면, $O(1)$만에 다음 해시 값 $H[1] = B*4 + C*2 + D*1$ 를 구할 수 있다.

다음 식으로 보면 이해가 좀 더 빠를 것이다.

$H[1] = (H[0] - A*4) * 2 + D$

이는 상수시간에 동작하는 연산이다. 즉 `"ABC"`에 대한 매칭을 수행하고, 다음 문자열인 `"BCD"`에 대한 매칭을 수행할 때 상수시간이 동작한다는 의미이다.

이 경우 시간 복잡도는 $O(N)$이 된다. ($N$은 문자열의 길이)

이해가 안되는 부분은 코드를 보며 극복해보자.

### C++ 구현 코드

```cpp
vector<int> RabinKarp (string &T, string &P) {
    vector<int> results;
    int p_length = P.size();
    int t_length = T.size();
    
    lint p_hash = 0, t_hash = 0, head = 1; // head는 가장 첫 원소에 대한 해시값을 뺄 때 사용한다.
    for(auto &x: P) {					   // 즉, p^(p_length-1)이 된다.
        p_hash = ((p_hash * p) + x) % MOD;
    } // 패턴의 해시값을 먼저 구한다.
    for(int i=0; i<p_length; i++) {
        t_hash = ((t_hash * p) + T[i]) % MOD;
        if(i) head = head * p % MOD;
    } // 가장 첫 해시값 즉, H[0]을 구한다.
    
    for(int i = 0; i <= t_length-p_length; i++) {
        if(p_hash == t_hash) 
            results.emplace_back(i+1);
        
        t_hash -= T[i] * head % MOD;
        t_hash = (t_hash * p + T[i+p_length]) % MOD;
        
        if(t_hash < 0) t_hash += MOD;
    }
    
    return results;
}
```

## KMP(Knuth-Morris-Prett) Algorithm

라빈 카프가 해시값을 통해 하나씩 건너가면서 보는 시간을 단축했다면, **KMP**는 하나씩 건너가지 않는다.

KMP에서 도입한 개념은 **실패 함수 (failure function)**이다.

이 실패함수는,

**'문자 매칭에 실패했을 때, 얼만큼 건너뛰어야 하는가?'**를 알기 위해 사용된다.

다른 말로는,

**'문자 매칭에 실패하기 직전 상황에서, 접두사 / 접미사가 일치한 최대 길이'**라고 풀이된다.

`"ABCDABCDABEE"` 에서 `"ABCDABE"` 를 찾는 경우를 살펴보도록 하자.

![](https://velog.velcdn.com/images/dnr6054/post/0d911604-fdcf-41c6-9690-c5fb67cf7166/image.png)

> 자 여기서 이제 C와 E가 다르므로 다시 Text의 두 번째 인덱스부터 검색한다? **That's no no**

`"ABCDAB"` 까지는 맞은 것이기 때문에 여기 AB로 시작하는 다섯 번째 인덱스부터 비교하면 된다.

심지어 다섯 번째, 여섯 번째 인덱스는 같다는 것이 보장되어 있기 때문에 일곱 번째 인덱스부터 비교하면 된다.

> 그러면 문자열 검색을 $O(N+M)$ 에 할 수 있게 된다! ($N$, $M$은 각각 `Text`와 `Pattern`의 길이)

이게 바로 **KMP**의 아이디어다.

그리고 $i$ 인덱스에서 틀렸을 때, $pi(i)$ 부터 다시 보면 돼요! 라고 알려주는 것이 바로 실패 함수의 역할인 것이다. 

> 다시 한번 짚고 넘어가자.
>
> 실패함수는 **'문자 매칭에 실패하기 직전 상황에서, 접두사 / 접미사가 일치한 최대 길이'** 이다.
>
> `"ABCDADE"` 를 예로 들면
> 
> - `"A"` : $pi(0) = 0$ (없음)
> - `"AB"` : $pi(1) = 0$ (없음)
> - `"ABC"` : $pi(2) = 0$ (없음)
> - `"ABCD"` : $pi(3) = 0$ (없음)
> - `"ABCDA"` : $pi(4) = 1$ (`"A"`)
> - `"ABCDAB"` : $pi(5) = 2$ (`"AB"`)
> - `"ABCDABE"` : $pi(6) = 0$ (없음)

### 알고리즘 동작 과정

실패함수 `pi` 가 구현되어 있다는 가정하에 한번 구현 과정을 살펴보자.

`Text`(`T`, 길이 $n$)에서 `Pattern`(`P`, 길이 $m$)를 찾아보자.

1. 우선 `i` 가 `T`를 훑게 하자. `j`는 일단 0으로만 두자.
> ```cpp
> for(int i=0,j=0; i<n; i++){
> ```
2. 그 다음, 루프 안에서 `T[i]` 와 `P[j]` 가 다르다면 같아지거나 `j` 가 `0` 이 될 때까지 계속 실패 함수를 이용해서 점프를 한다. 
> ```cpp
> while(j>0 && T[i] != P[j]) j = pi[j-1];
> ```
3. `T[i]` 와 `P[j]` 가 같다면 둘 중 하나다.
	1. 만약 `j`가 `m-1` 즉, 패턴의 끝까지 일치한다면 정답 벡터에 그 위치를 넣는다. 그리고 j는 다시 실패함수를 이용해 움직인다.
    > ```cpp
    > result.push_back(i-M+2);
    > j = pi[j];
    > ```
    2. 아니라면 j를 1 증가시켜서 다음 문자도 일치하는지 확인한다.
    
정리하면 다음과 같다.

```cpp
vector& kmp(string T, string P) {
	int n = T.size();
    int m = P.size();
    vector<int> result;
    
    for(int i=0,j=0; i<n; i++) {
    	while(j>0 && T[i] != P[j])
        	j = pi[j-1];
        if(T[i] == P[j]) {
        	if(j == m-1) {
            	result.push_back(i-m+2);
                j = pi[j];
            }
        } else {
        	j++;
        }
    }
    
    return result;
}
```

### 실패함수 구현 방법

자 이제 남은 건 하나다. 실패함수 `pi` 는 도대체 어떻게 구현할까?

Naive하게 접근한다면, 접두사의 길이를 증가시키면서 접미사의 길이를 증가시키고 비교해야 하므로 $O(M^2)$ 임을 알 수 있다.

말도 안된다. 배보다 배꼽이 더 큰 상황이다. 

하지만? 여기에서 위에 쓴 **KMP**의 구현을 가져다가 써볼 수 있다

```cpp
	for(int i=1,j=0; i<n; i++) {
    	while(j>0 && T[i] != P[j])
        	j = pi[j-1];
        if(T[i] == P[j]) {
        	pi[i] = ++j;
        }
    }
```

굉장히 유사하다! 

그럼 한번 pi 배열의 생성과정을 살펴보도록 하자.

`pi[0]`은 0으로 두고 `pi[1]`부터 계산하자.

`P[1]` 과 `P[0]` 은 다르므로 `pi[1]` 은 0이 된다.

| | | | | | | | | | |
|-|-|-|-|-|-|-|-|-|-|
|`pi`| index | 0 | 1 | 2 | 3 | 4 | 5 | 6 |   |
|    | value | 0 | 0 |   |   |   |   |   |   |
|  P | index | 0 | 1 | 2 | 3 | 4 | 5 | 6 |   |
|    | value | A | B | C | D | A | B | E |   |
|  P | index |   | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
|    | value |   | A | B | C | D | A | B | E |

쭉쭉쭉 다 다르므로 `pi[3]`까지 0이 저장된다.

| | | | | | | | | | | | |
|-|-|-|-|-|-|-|-|-|-|-|-|
|`pi`| index | 0 | 1 | 2 | 3 | 4 | 5 | 6 |   |   |   |
|    | value | 0 | 0 | 0 | 0 |   |   |   |   |   |   |
|  P | index | 0 | 1 | 2 | 3 | 4 | 5 | 6 |   |   |   |
|    | value | A | B | C | D | A | B | E |   |   |   |
|  P | index |   |   |   | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
|    | value |   |   |   | A | B | C | D | A | B | E |

이제 `P[4]`와 `P[0]`은 같으므로 드디어 `pi[4]` 에 1이 저장된다.

| | | | | | | | | | | | | |
|-|-|-|-|-|-|-|-|-|-|-|-|-|
|`pi`| index | 0 | 1 | 2 | 3 | 4 | 5 | 6 |   |   |   |   |
|    | value | 0 | 0 | 0 | 0 | 1 |   |   |   |   |   |   |
|  P | index | 0 | 1 | 2 | 3 | 4 | 5 | 6 |   |   |   |   |
|    | value | A | B | C | D | A | B | E |   |   |   |   |
|  P | index |   |   |   |   | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
|    | value |   |   |   |   | A | B | C | D | A | B | E |

그다음은 `P[5]`와 `P[1]`을 비교한다. 같으므로 `pi[5]`에는 2가 저장된다.

| | | | | | | | | | | | | |
|-|-|-|-|-|-|-|-|-|-|-|-|-|
|`pi`| index | 0 | 1 | 2 | 3 | 4 | 5 | 6 |   |   |   |   |
|    | value | 0 | 0 | 0 | 0 | 1 | 2 |   |   |   |   |   |
|  P | index | 0 | 1 | 2 | 3 | 4 | 5 | 6 |   |   |   |   |
|    | value | A | B | C | D | A | B | E |   |   |   |   |
|  P | index |   |   |   |   | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
|    | value |   |   |   |   | A | B | C | D | A | B | E |

다시 다르므로 `j`를 옮긴다. 하지만 `pi[1]`이 0이므로 `j`가 0으로 된다.

`P[6]`와 `P[0]`이 다르므로 다시 `pi[6]`도 0으로 저장된다.

| | | | | | | | | | | | | | | |
|-|-|-|-|-|-|-|-|-|-|-|-|-|-|-|
|`pi`| index | 0 | 1 | 2 | 3 | 4 | 5 | 6 |   |   |   |   |   |   |
|    | value | 0 | 0 | 0 | 0 | 1 | 2 | 0 |   |   |   |   |   |   |
|  P | index | 0 | 1 | 2 | 3 | 4 | 5 | 6 |   |   |   |   |   |   |
|    | value | A | B | C | D | A | B | E |   |   |   |   |   |   |
|  P | index |   |   |   |   |   |   | 0 | 1 | 2 | 3 | 4 | 5 | 6 |
|    | value |   |   |   |   |   |   | A | B | C | D | A | B | E |

이렇게 실패함수가 구현된다.

전체 코드는 다음과 같을 것이다.

### C++ 구현 코드

```cpp
vector<int>& KMP(string T, string P) {
	int n = T.size();
    int m = P.size();
    vector<int> result;
    
    for(int i=1,j=0; i<n; i++) {
    	while(j>0 && T[i] != P[j])
        	j = pi[j-1];
        if(T[i] == P[j]) {
        	pi[i] = ++j;
        }
    }
    
    for(int i=0,j=0; i<n; i++) {
    	while(j>0 && T[i] != P[j])
        	j = pi[j-1];
        if(T[i] == P[j]) {
        	if(j == m-1) {
            	result.push_back(i-m+2);
                j = pi[j];
            }
        } else {
        	j++;
        }
    }
    
    return result;
}
```

## Boyer-Moore-Horspool Algorithm

> 사실 이 알고리즘은 알고리즘 문제를 풀기에 적합하지 않다. 
>
> 최악의 경우 시간복잡도가 $O(NM)$ 까지 뛰기 때문이다. 그리고 대부분 알고리즘 문제는 이런 케이스를 포함한다.
>
> 하지만 위와 같은 경우는 실생활에서 드물고 대부분의 경우 **KMP**보다 좋은 성능을 보이기 때문에 가장 널리 쓰이는 알고리즘이다.

`"TRUSTHARDTOOTHBRUSH"` 에서 `"TOOTH"` 를 검색한다고 해보자.

### Bad Match Table

우선 Bad Match Table을 만들어야 한다. 

일단 이게 뭔지는 알고리즘 설명에서 보도록 하고 어떻게 만드는지부터 살펴보자.

기본적으로 Bad Match Table은 다음과 같은 식을 반복하여 만들어진다.

> **value = length - index - 1, (every other characters: length)**

```
T O O T H
0 1 2 3 4
```

에 대해서 Bad Match Table은 다음과 같은 과정으로 생성된다.

- `'T'` (5 - 0 - 1)

| Letter | T | O | H | * |
|--------|---|---|---|---|
| Value  | 4 |   |   | 5 |

- `'O'` (5 - 1 - 1)

| Letter | T | O | H | * |
|--------|---|---|---|---|
| Value  | 4 | 3 |   | 5 |

- `'O'` (5 - 2 - 1)

| Letter | T | O | H | * |
|--------|---|---|---|---|
| Value  | 4 | 2 |   | 5 |

- `'T'` (5 - 3 - 1)

| Letter | T | O | H | * |
|--------|---|---|---|---|
| Value  | 1 | 2 |   | 5 |

- `'H'` 마지막 글자는 그 전에 값이 지정되지 않았다면 문자열의 길이로 저장한다.

| Letter | T | O | H | * |
|--------|---|---|---|---|
| Value  | 1 | 2 | 5 | 5 |

코드는 다음과 같을 것이다.

```cpp
vector<int>& BadMatchTable(string P) {
	vector<int> result(26,P.size()); // 대문자 알파벳만 등장한다고 가정하자.
	for(int i=0; i<P.size()-1; i++) {
    	result[P[i]-'A'] = P.size()-i-1;
    }
    return result;
}
```

### 알고리즘 동작 과정

자 그러면 이제 어떻게 탐색을 할 것이냐.

```
T: T R U S T H A R D T O O T H B R U S H
P: T O O T H
```

여기에서 P의 가장 마지막 글자와 T의 다섯 번째 글자를 비교한다. 

다르다! 

그렇기 때문에 P를 이동시키는데, 이 때, T[4]의 Bad Match Table 값 만큼 점프를 한다.

여기에서는 1이므로 한 칸 이동한다.

```
T: T R U S T H A R D T O O T H B R U S H
P:   T O O T H
```

H 끼리 같고, T도 같다. S는 Bad Match Table에서 정의되지 않았으므로 패턴의 길이인 5가 된다.

따라서 이번에는 다섯 칸을 점프한다.

```
T: T R U S T H A R D T O O T H B R U S H
P:             T O O T H
```

O는 Bad Match Table에서 2로 정의되어 있으므로 두 칸 점프한다.

```
T: T R U S T H A R D T O O T H B R U S H
P:                 T O O T H
```

다시 T는 Bad Match Table에서 1로 정의되어 있으므로 한 칸 점프한다.

```
T: T R U S T H A R D T O O T H B R U S H
P:                   T O O T H
```

이런식으로 Boyer-Moore-Horspool 알고리즘은 점프를 한번에 어마어마하게 많이 뛴다. 

> 문자열에서 건너뛸 수 있는 만큼은 모두 건너뛴다

구현은 다음과 같이 할 수 있을 것이다.

### C++ 구현 코드

```cpp
vector<int>& BadMatchTable(string P) {
	vector<int> result(26,P.size()); // 대문자 알파벳만 등장한다고 가정하자.
	for(int i=0; i<P.size()-1; i++) {
    	result[P[i]-'A'] = P.size()-i-1;
    }
    return result;
}

vector<int>& BoyerMooreHorspool(string T, string P) {
	vector<int> bad_match = BadMatchTable(P);
    
    vector<int> result;
    
    int s = 0;
    int n = T.size(), m = P.size();
    
    while(s <= n-m) {
    	int j = m-1;
        while(j>=0 && P[j] == T[s+j]) j--;
        if(j < 0) {
        	result.push_back(s);
            s += (s+m < n) ? m - bad_match[T[s+m] - 'A'] : 1;
        } else {
        	s += max(1, j - bad_match[T[s+j] - 'A']);
        }
    }
    
}
```

사실 이 방법은 보이어-무어 알고리즘을 호스풀 교수가 간략하게 바꾼 것이다.

보이어-무어 알고리즘은 성능이 뛰어나기 때문에 GNU grep을 비롯한 여러 곳에 사용되며 문자열 검색 알고리즘 성능 비교의 표준으로 쓰이지만, 비교적 복잡하다는 단점이 있다.

그것을 가장 중요한 부분만 남긴 것이 바로 이 알고리즘인 것이다.
