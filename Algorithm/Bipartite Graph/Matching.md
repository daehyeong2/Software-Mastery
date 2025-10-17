매칭(matching)이란 무엇일까?
-> 매칭이란 그래프 G = (V, E)에 대해서, 부분집합 M⊆E의 모든 간선이 정점을 공유하지 않을 때, M을 matching이라고 부른다.

즉, 각 정점은 각각 간선을 1개씩만 가진다는 것이다.

![[Pasted image 20251017135913.png]]
위 이미지에서 모든 선분은 간선이고, 빨간 선분이 matching이다. 이미지를 보면 모든 정점은 한 개의 빨간 선(matching)만 연결되어 있는 것을 알 수 있다. 이런 식으로 **한 정점에 대하여 여러 간선이 연결되지 않도록 연결짓는 선들의 집합을 매칭(matching)** 이라고 하는 것이다.

> Perfect Matching이란?
> -> 모든 정점이 matching되는 상태

그럼 우리는 매칭과 관련해서 어떤 작업을 시도할 수 있을까? 우리는 매칭의 최대/최소를 구해볼 수 있을 것이다.

하지만 매칭의 최소를 구하는 것은 선분 한 개만 연결하면 되는 것이기에 어렵지 않다.
따라서 matching의 최대를 구하는 문제를 다음과 같이 정의한다.

>**"Maximum matching problem"**
>어떤 그래프가 주어졌을 때 크기가 가장 큰 matching을 찾으시오.

최대 매칭을 구하기 위해 우리는 **이분 매칭**이라는 알고리즘을 활용할 수 있다.

번외로, matching과 자주 함께 언급되는 개념으로 Vertex Cover라는 것이 있다.
이는 그래프 G를 전부 커버할 수 있는 정점의 집합이라는 의미인데, 이 또한 따로 정리할 예정이다.


참고: [https://gazelle-and-cs.tistory.com/12](https://gazelle-and-cs.tistory.com/12) [Gazelle and Computer Science:티스토리]