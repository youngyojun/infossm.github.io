

---

layout: post 

title: "Union Find의 시간 복잡도 증명" 

author: junis3

date: 2021-04-19

tags: []

---



## Union Find

Union Find 또는 Disjoint Set이라고 불리는 자료구조는 여러 개의 원소를 포함하고 있는 집합들을 다루는 자료구조이다. 초기에, $N$개의 원소들과 $N$개의 집합들이 만들어져 있고, 각각의 집합은 서로 다른 원소 하나씩을 가지고 있다. 이 위에 다음과 같은 두 개의 연산을 할 수 있다.

1. Union(x, y) : 원소 x가 속한 집합과, 원소 y가 속한 집합을 합친다.
2. Find(x) : 원소 x가 어떤 집합에 속해있는지 반환한다. 보통 x가 속해있는 집합의 대표 원소 하나를 반환한다.

```c++
struct dsu {
    dsu() { iota(p, p+maxn, 0); }
    int p[maxn];
    int root(int k) { 
		    if (p[k] != k) return root(p[k]);
		    else return k;
    }
    bool connect(int x, int y) {
        x = root(x); y = root(y);
        if (x==y) return false;
        p[y] = x;
        return true;
    }
};
```

이 두 연산은 보통 위와 같이 집합을 **트리**에 대응시켜 생각하는 방법으로 구현하는데, 가장 쉬운 방법으로 구현하면 최악의 경우 각 연산을 수행하는 데에 $O(N)$의 시간이 소요된다. 트리가 균형잡혀있지 않고 한 쪽으로 쏠려있을 수 있기 때문이다. 이를 개선하기 위해 두 가지 최적화를 도입한다.

1. Union by Rank : 각 집합의 'rank'를, 더 큰 집합일수록 더 큰 rank를 가지도록 매긴다. 그리고 union 연산에서 언제나 더 작은 집합을 더 큰 집합에 합친다.
2. Path Compression : Find(x)는 x에서 트리의 루트까지의 정점들을 차례대로 방문하는 형태로 구현한다. 경로 상의 모든 정점들을 모두 루트 정점을 바로 가리키게 한다.

두 최적화를 적용하면, 연산당 평균 (amortized) 시간 복잡도는 $O(\alpha(N))$ 까지 줄어든다. 이 때 $\alpha$ 함수는 아주 느리게 증가하기로 유명하기 때문에 사실상 $O(1)$ 나 다름 없다. 이들 최적화를 모두 구현하면 다음과 같다:

```c++
struct dsu {
    dsu() { iota(p, p+maxn, 0); }
    int p[maxn], rank[maxn];
    int root(int k) { 
		    if (p[k] != k) return p[k] = root(p[k]);
		    else return k;
    }
    bool connect(int x, int y) {
        x = root(x); y = root(y);
        if (x==y) return false;
        if (rank[x] < rank[y]) swap(x, y);
        if (rank[x] == rank[y]) rank[x]++;
        p[y] = x;
        return true;
    }
};
```

...까지가 우리가 알고 있는 사실이다. 이 글에서는 이 사실들 자체에 대해서는 다루지 않는다. 만약 위 사실들이 불명확하게 느껴지면 [위키백과](https://ko.wikipedia.org/wiki/%EC%84%9C%EB%A1%9C%EC%86%8C_%EC%A7%91%ED%95%A9_%EC%9E%90%EB%A3%8C_%EA%B5%AC%EC%A1%B0) 등을 참조하라. 이 사실이 어떤 이유에서 성립하는지, 그리고 둘 중 하나의 최적화만 적용하면 시간 복잡도가 어떻게 되는지는 잘 알려져 있지 않다.



## Union by Rank

먼저, Rank 최적화부터 살펴보자. Rank 최적화는 트리가 균형잡혀있게 강제한다. 각 집합에 대해, '이 집합을 나타내는 트리의 높이'를 rank라는 변수에 저장한다. 그리고 두 집합을 합칠 때에는 rank가 작은 집합을, rank가 큰 집합 아래에 붙이는 형태로 구현한다.

또는, 각 집합에 대해 rank 변수를 유지하는 대신 size 변수를 유지해 같은 방법으로 최적화하는 Union by size 기법도 있다. 이 방법은 'smaller-to-larger trick'이라는 이름으로 일반적인 트리 알고리즘의 최적화에 쓰이곤 한다. 두 알고리즘 중 어떤 것이든 적용했을 때, 연산당 시간 복잡도는 $O( \log N)$ 으로 줄어든다.



**보조정리 1.** 정점 x가 속한 트리가 바뀌면, (rank 최적화를 수행할 경우) x의 rank는 언제나 1 이상 증가하고, (size 최적화를 수행할 경우) size는 언제나 두 배 이상 증가한다.

**증명.** 정점 x가 속한 트리가 바뀐다는 뜻은, x를 rank(size)가 더 크거나 같은 정점 y 아래에 붙인다는 것이다. 

* rank[y]는 rank[x] 이상이므로, 합친 이후의 rank[x]는 rank[x] = rank[y]일 경우 rank[x]+1이거나 그렇지 않을 경우 rank[x]가 된다. 두 경우 모두, 합친 이후의 rank[x]는 rank[y]+1 이상이다.
* size[y]는 size[x] 이상이므로, 이므로 합친 이후의 size[x]인 size[x] + size[y]는 2 size[x] 이상이다.

더불어, 정점 x가 속한 트리가 바뀔 때마다 정점 x의 깊이는 1씩 증가한다는 사실을 알 수 있다.



**보조정리 2.** 모든 정점의 rank는 $\lceil \log_2 N \rceil$ 이하이고, 모든 정점의 size는 $N$ 이하이다.

**증명.** 모든 정점의 size가 $N$ 이하인 것은 정점의 개수가 $N$개이므로 자명하다. 그리고,

- size가 1일 때, rank는 1.
- size가 3 이하일 때, rank는 2 이하.
- size가 7 이하일 때, rank는 3 이하.
- size가 15 이하일 때, rank는 4 이하.
- ...

임이 귀납적으로 증명 가능하다. 따라서, 모든 정점의 rank는 $\lceil \log_2 N \rceil$ 이하이다.



위 두 보조정리로 인해, union by rank와 union by size의 시간 복잡도가 동시에 증명된다. Union by size를 사용한다고 하자. 어떤 정점 x에 대해, x가 속한 트리가 바뀔 때마다 size가 2배 이상 증가한다. 그런데, size가 $N$ 초과가 되는 것은 불가능하므로, x가 속한 트리가 바뀌는 횟수는 많아야 $\lceil \log_2 N \rceil$ 번이다. 그런데, 정점 x가 속한 트리가 바뀔 때마다 정점 x의 깊이는 1씩 증가한다. 초기에 모든 정점들은 자신이 속한 트리의 루트였으므로 깊이가 0이었다. 따라서 연산들을 수행한 이후에도 각 정점의 깊이는 $\lceil \log_2 N \rceil$ 이하이다. 따라서 find 함수는 언제나 $\lceil \log_2 N \rceil$ 개 이하의 정점을 순회하게 되고, $O( \log N)$ 시간에 작동한다.

Rank/Size compression만을 사용해도 트리의 균형이 보장된다! 이 성질은 KOI 지역예선 필기 시험에서도 물은 적이 있고, Heavy Light Decomposition 등 다양한 알고리즘의 시간 복잡도를 보장해주기도 하므로, 꼭 기억해두면 좋다.



## Path Compression

다음, 경로 압축 (Path compression)은 Find(x)를 실행하는 과정 가운데에서 경로 상의 모든 정점들을 곧바로 루트 정점 아래에 달아주는 최적화이다. 결론부터 이야기하면, 경로 압축만을 적용했을 때, 연산당 평균 시간 복잡도는 $O( \log N )$이 된다. 이에 대한 증명은 지금은 하지 않지만, 아래 Union by Rank + Path compression의 시간 복잡도가 연산당 평균 $O(\log^\star N)$이라는 사실에 대한 증명을 먼저 읽어보라. 그 이후에 증명에 대한 간략한 요약을 서술한다.

요지는, 경로 압축만을 사용했을 때, 각각의 연산은 최대 $O(N)$까지의 시간이 걸릴 수 있어 시간 복잡도가 보장되지 않는 것처럼 느껴질 수 있지만, $M$번의 Find 연산을 수행하는 데에 $O(M \log N)$의 시간만이 소요됨이 증명 가능하므로, 안심하고 사용할 수 있다는 것이다.



## Union by Rank + Path Compression

앞서 Union Find를 최적화하는 두 가지 방법의 시간 복잡도에 대해서 알아보았다. 둘 모두를 적용했을 때에는, 연산당 평균 $O(\alpha (N))$ 이라는 시간 복잡도가 도출된다. $\alpha(N)$ (Ackermann 역함수)는 매우 빠르게 증가하는 함수인 애커만 함수로부터 정의되는데, 함숫값은 다음과 같다.

- $1 \le N < 3 = 2 + 4 - 3$ 일 때, $\alpha(N) = 1$.
- $3 \le N < 7 = 2 \times 5 - 3$ 일 때, $\alpha(N) = 2$.
- $7 \le N < 63 = 2^6 - 3$ 일 때, $\alpha(N) = 3$.
- $63 \le N < 2^{2^{2^{2^{2^{2^2}}}}}-3 = 2^{2^{2^{65536}}} - 3$ 일 때, $\alpha(N) = 4$.

즉, 우주에 있는 원자 개수보다 작은 모든 정수 값에 대해, $\alpha(N) \le 4$ 이다. Ackermann 역함수는 계산 복잡도에서 흔히 쓰이는 함수들 가운데에 가장 느리게 증가하는 함수이다. 

조금 더 빠르게 증가하는 함수 $\log^\star$ 를 살펴보자. $\log^\star N$ 은, $N \leftarrow \log_2 N $ 연산을 반복해서 $N \le 1$ 이 되도록 하는 연산 횟수로 정의된다. 함숫값은 다음과 같다.

- $1 < N \le 2^1 = 2$ 일 때, $\log^\star N = 1$.
- $2 < N \le 2^2 = 4$ 일 때, $\log^\star N = 2$.
- $4 < N \le 2^4 = 16$ 일 때, $\log^\star N = 3$.
- $16 < N \le 2^{16} = 65536$ 일 때, $\log^\star N = 4$.
- $65536 < N \le 2^{65536}$ 일 때, $\log^\star N = 5$.

우주에 있는 원자 개수보다 작은 모든 정수 값에 대해 $\log^\star N \le 6$ 이며, $\alpha(N) \le 4$ 를 만족하는 범위 안에 있는 $N$에 대해, 또한 $\log^\star N \le 6$이다. $\log^\star$이 $\alpha$보다 조금 더 빠르게 증가하기는 하지만, 실제 구현 상에서 상수와 다를 바 없다는 것은 동일하다. 두 최적화를 모두 했을 때 연산당 평균 $O( \alpha(N))$ 의 시간 복잡도가 도출된다는 사실은 1989년에 밝혀졌고, 그 이전까지 알려진 시간 복잡도의 상한은 평균 $O( \log^\star N)$이었다. 이 글에서는, 두 최적화를 모두 적용했을 때 연산당 평균 $O(\log^\star N)$의 시간이 소요됨을 증명한다.



**증명.** 경로 압축이 있든 없든, 트리의 루트, 정점의 랭크, 그리고 트리 안에 있는 정점들은 그대로이다. 루트였던 정점 x가 다른 정점의 자식으로 합쳐질 때에도, rank[x]는 원래의 값을 유지한다고 가정하자. 이 때, 두 개의 트리가 합쳐지는 방법은 *트리 A의 루트가, 트리 B의 루트 바로 아래에 붙는* 방법밖에 없음에 주목하자. 정점 x가 루트 정점이 아니게 되는 순간, rank[x]는 더 이상 변할 일이 없음 또한 분명하다.



**보조정리 3.** x가 루트 정점이 아니고, 부모 p가 존재할 때, rank[x] < rank[p]이다.

**증명.** 정점 압축이 없었다고 가정해 보자. 그러면 정점 x가 정점 p 아래에 붙여지는 모양으로 합쳐지는 순간이 있었을 것이다. 이 때, i) 원래부터 rank[x] < rank[p]였거나, ii) 원래 rank[x] = rank[p]였더라도 합쳐지는 순간 rank[p]가 증가해서 rank[x] < rank[p]가 되었을 것이다. 따라서 rank[x] < rank[p]이고, 언제나 부모는 자식보다 큰 rank를 가진다. 즉, 언제나 조상은 자손보다 큰 rank를 가진다. 그런데 경로 압축 이후 x의 부모가 될 수 있는 정점들은 원래 x의 조상이었을 것이다. 따라서, 경로 압축이 있었더라도 언제나 rank[x] < rank[p]가 성립한다.



**보조정리 4.** 경로 압축으로 인해 x의 부모 정점이 바뀌면, 바뀐 부모 정점의 rank는 원래 부모 정점의 rank보다 1 이상 크다.

**증명.** 앞서 언급했듯, 언제나 조상은 자손보다 큰 rank를 가진다. 바뀐 부모 정점은 x가 속한 트리의 루트였을 것이므로, 원래 부모 정점의 조상이었을 것이다.



앞서 Union by Rank의 시간 복잡도 증명에서 사용했듯, 루트의 rank가 $k$인 트리는 $2^k$개 이상의 정점을 가지고 있다. 따라서, 각 정수 $k$에 대해, rank가 $k$인 정점은 많아야 $N/{2^k}$ 개 존재한다. 이 때, (정점 개수가 우주의 원자 개수보다 적다고 가정하고) 정점들을 다음과 같이 6개의 바구니로 나누어보자. 일반적으로는 $\log^\star N$ 개의 바구니가 생기게 될 것이다.

1. Rank = 0인 정점
2. Rank = 1인 정점
3. Rank = 2, 3인 정점
4. Rank가 4 이상 $15 = 2^4-1$ 이하인 정점
5. Rank가 16 이상 $65535 = 2^{16} - 1$ 이하인 정점
6. Rank가 65536 이상 ($2^{65536} - 1$ 이하)인 정점

여기서, $2N / 2^x > N / 2^x + N / 2^{x+1} + \cdots + N / 2^{2^x-1}$ 이기 때문에, Rank가 $[x, 2^x-1]$ 범위에 있는 정점은 많아야 ${2N} / 2^x$ 개 존재한다. 이제 총 $M$번의 Find 연산이 일어났다고 가정하자. 각각의 Find 연산에서는 해당 정점에서 루트로 가는 하나의 경로를 순회했을 것이고, 그 경로를 이루는 간선들은



1. 어떤 정점에서 루트로 곧바로 가는 간선 (이런 간선은 2, 3에서 세지 않는다)
2. 다른 바구니에 속하는 두 정점을 잇는 간선
3. 같은 바구니에 속하는 두 정점을 잇는 간선



으로 분류된다. 예를 들어, 한 번 Find 연산을 실행한 정점에서 곧바로 Find 연산을 다시 실행하면, 경로 압축이 이미 일어났으므로 곧바로 루트 정점을 찾을 것이며, 1.에 속하는 간선 하나만을 방문하게 될 것이다. 이제, 각 종류의 간선들을 방문하는 횟수를 세 보자. 물론, 각 종류의 간선을 방문하는 횟수가 각각 $O((N+M) \log^\star N)$ 이하라고 주장할 예정이다.



1. 루트로 향하는 간선은 한 번의 Find 연산마다 한 번씩만 방문할 수 있다. 총 $O(M)$개.
2. **보조정리 1**에 의해, 루트로 올라가는 경로 위에서, Rank는 단조적으로 증가한다. 바구니의 개수가 $\log^\star N$개이므로, 한 경로 위에서 방문할 수 있는 바구니의 개수도 $O(\log^\star N)$ 개이다. 총 $O(M \log^\star N)$ 개.
3. 여기 속하는 각 간선들은 $v \rightarrow p$ ($p$는 $v$의 부모 정점)의 꼴일 것이다. 정점 $v$와, 정점 $v$가 속하는 바구니 $[x, 2^x-1]$를 고정시키고, $p$의 역할을 하는 정점들의 모음을 생각해 보자. 먼저, $p$가 트리의 루트일 경우는 여기서 세지 않으므로, $p$의 역할을 하는 정점들은 매번 바뀐다. (간선을 방문할 때마다 루트에 곧바로 붙여버리니..) **보조정리 2**에 의해, $v$의 부모 정점의 Rank는 부모가 바뀔 때마다 1 이상 증가하고, 따라서, $v$에 대해 가능한 $p$의 개수는 바구니의 Rank의 가짓수인 $2^x$개 이하이다. 바구니 $[x, 2^x-1]$에 속하는 정점 개수는 $N/2^x$개 이하이므로, 가능한 $v$의 가짓수는 $N / 2^x$ 개이다.
   따라서, 바구니 $[x, 2^x]$ 안에 있을 수 있는 간선 $v \rightarrow p$ 의 가짓수는 $2^x \times N/2^x = O(N)$개 이하이고, 바구니 개수가 $O(\log^\star N)$개이므로 3.에 속하는 총 간선의 가짓수는 $O(N \log^\star N)$개이다.



따라서, $M$번에 걸친 Find 연산에서 "$v \rightarrow$ 루트" 로의 경로를 이루는 간선의 가짓수의 총합은 $O(M) + O(M \log^\star N) + O(N \log^\star N) = O((M+N)\log^\star N)$ 개이고, $M$번의 Find 연산의 총 시간 복잡도 또한 $O((M+N) \log^\star N)$이다. 따라서 연산 하나의 평균 시간 복잡도는 $O(\log^\star N)$. 증명 끝.



증명의 내용에서도 볼 수 있듯, 증명에서 변수들의 범위를 아주 빡빡하게 사용하지는 않았다. 1989년 Fredman과 Saks는 위 증명을 넘어서서 Find 연산 하나당 평균 $O(\alpha(N))$의 시간 복잡도에 실행됨을 보였다. 또한 이렇게 상수에 가까운 시간에 작동하는 것이 가능함에도 불구하고 Find 연산을 "진짜" 상수 시간에 하는 것은 불가능함 또한 증명되었다.

다시 Union by rank가 없는 Path compression으로 돌아와서, 위 증명과 유사한 방법으로 Path compression만 사용했을 때의 시간 복잡도를 증명할 수 있으리라 생각할 수도 있다. 하지만 이 경우에는 rank가 $k$인 정점은 많아야 $N/{2^k}$ 개 존재한다는 사실을 이용할 수 없다. 따라서 rank를 증명에 사용할 수 없지만, size는 여전히 사용할 수 있기 때문에 이를 이용해 시간 복잡도의 상한을 증명할 수 있다. 자세한 증명은 스스로 해 보기를 권장한다.

이 글에 걸쳐 Union-Find 자료구조의 시간 복잡도를 증명해 보았다. 둘 중 하나의 최적화만을 적용해도 아주 빠르게 작동함만이 알려져 있어서 모두가 안심하고 사용해오던 자료구조였을텐데, 이렇게 시간 복잡도의 상한이 바구니 방식으로 증명되는 것을 보니 새로웠을 것이다.