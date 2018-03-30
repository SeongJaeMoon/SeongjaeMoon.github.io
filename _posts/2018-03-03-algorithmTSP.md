---
layout: post
title:  "여행하는 외판원 문제"
date:   2018-03-03 16:00:00
author: Seongjae Moon
categories: Algorithm
tags:   여행하는외판원문제 알고리즘 Traveling-Salesman-Problem Java 완전탐색 최적화문제 재귀함수
---

한 주의 마무리를 알리는 알고리즘 문제 풀이 포스팅! 요 근래엔 나의 생일을 맞이해서 참 많은 일들이 있었다. 여러가지 기분 좋은 일들이 많았던 한 주인 것 같다. 늘 이렇게만 가즈아!!! 거두절미하고, 오늘은 완전탐색 최적화 문제에 대해서 알아보자.

책에서는 C++를 이용해서 코딩하지만, 스터디를 위해 사용하기로 한 언어가 JAVA이기 때문에 코드는 JAVA로 작성되었으며, 책의 내용을 나름대로 재구성하여 작성하였다.

### 먼저, 여행하는 외판원 문제(TSP)에 대해서 알아보자.
우선, 여행하는 외판원 문제는 최적화 문제이다. 최적화 문제란 앞서 살펴 봤던, [게임판 덮기](https://seongjaemoon.github.io/algorithm/2018/02/18/algorithmBoardCover.html)나 [소풍](https://seongjaemoon.github.io/algorithm/2018/02/09/algorithmPicnic.html)같은 문제와는 달리 문제의 답이 하나가 아니라 여러 개이고, 그 중에서 어떤 기준에 따라 가장 **'좋은'** 답을 찾아 내는 문제이다. 문제의 이름에서 느껴지듯, 여행과 관련된 경로 계산 문제라는 것을 억지스럽게? 추측해볼 수 있다.

### 예를 들어보자.
![경로 찾기 예시](/assets/uploads/algorithm/tsp.png)
어떤 나라에 n(2<=n<=10)개의 큰 도시가 있다고 가정하자. 한 영업 사원이 한 도시에서 출발해 다른 도시들을 전부 한 번씩 방문한 뒤 시작 도시로 돌아오려고 한다.(문제를 간단히 하기 위해서, 각 도시들은 모두 직선 도로로 연결되어 있다고 가정한다.) 이때 영업 사원이 여행해야 할 거리는 어느 순서로 도시들을 방문하느냐에 따라서 달라지는데, 가능한 모든 경로 중 가장 짧은 경로를 찾아내는 문제라고 할 수 있다.   

위와 같이 도로마다 여러 도로망이 구성되어 있을 경우, 빨간색으로 색이 칠해진 경로의 길이에 합이 최소가 되게 만드는 문제라고 할 수 있다. 우리가 흔히 사용하는 지도 포털 사이트에서 경로를 찾아주는 알고리즘은 훨씬 더 복잡한 알고리즘을 사용하겠지만, 그 알고리즘에 기초가 되지 않을까 조심스럽게 생각해본다.

### 문제의 이해.
우선, 이런 경우의 수를 계산하는 문제를 풀기 위한 첫 번쩨 단계로 시간 안에 우리가 원하는 답을 구할 수 있을지 확인하는 절차가 필요하다. 이 문제에선 시작한 도시로 돌아오는 경로를 찾기 때문에, 경로의 시작점은 신경 쓰지 않고 무조건 0번 도시에서 출발한다고 가정해도 경로의 길이는 달라지지 않는다.(기준점) 때문에 남은 도시들을 어떤 순서로 방문할지를 결정하기만 하면 된다.

도시가 10개라면 경로를 찾아내는데 9!(362,880개)의 경우의 수를 계산해야 한다. 우리의 컴퓨터가 열일해야 될 것 같지만.. 컴퓨터는 상상 이상으로 빠른 연산이 가능하니 믿어보자. 그렇다면, 여행하는 외판원 코드를 살펴보자.
```java
import java.util.*;

public class TSP {
	//도시의 수
	private static int n;
	//두 도시 간의 거리를 저장하는 배열
	private static int dist[][];

	public static void main(String[] args) {
		Scanner sc = new Scanner(System.in);

		System.out.print("도시의 수?(2<=n<=10):");
		//도시의 수 입력.
		n = sc.nextInt();

		dist = new int[n][n];
		//거리를 랜덤으로 나타낸다.
		for(int i = 0; i < dist.length; ++i) {
			for(int j = 0; j < dist[i].length; ++j) {
				dist[i][j] = new Random().nextInt() + 1;
			}
		}
		//경로를 저장할 List 선언.
		List<Integer>path = new ArrayList<Integer>();
		//이미 들린 도시인지 확인하기 위한 배열 선언.
		boolean visited [] = new boolean[n];
		//첫 번째 도시를 시작점으로 한다.
		visited[0] = true;
		path.add(0);
		//각 도시간 경로의 길이를 표시.
		System.out.println(Arrays.deepToString(dist));
		//제일 짧은 경로의 합을 구해서 출력한다.
		System.out.println(shortestPath(path, visited, 0));

		sc.close();
	}
	//path: 지금까지 만든 경로
	//visited: 각 도시의 방문 여부
	//currentLength: 지금까지 만든 경로의 길이
	//나머지 도시들을 모두 방문하는 경로들 중 가장 짧은 것의 길이를 반환.
	private static int shortestPath(List<Integer> path, boolean visited[], int currentLength) {
		int len = path.size();
		//기저 사례: 모든 도시를 다 방문했을 때는 시작 도시로 돌아가고 종료한다.
		if(len == n) {
			return currentLength + dist[path.get(0)][path.get(len - 1)];
		}
		//임의의 매운 큰 값으로 초기화
		 int ret = 987654321;
		//다음 방문할 도시를 전부 시도한다.
		for(int next = 0; next < n; ++next) {
			//이미 방문했다면 조건 검사하지 않음.
			if(visited[next])continue;
			//path가 결정된 크기를 할당.
			int here = path.size() - 1;
			//경로에 인덱스를 할당.
			path.add(next);
			//방문 했음을 표시.
			visited[next] = true;
			//나머지 경로를 재귀 호출을 통해 완성하고 가장 짧은 경로의 길이를 얻는다.
			int cand = shortestPath(path, visited, currentLength + dist[here][here]);
			//경로의 값이 더 짧은 값을 선택한다.
			ret = Math.min(ret, cand);
			//다음 재귀호출시 방문할 경우의 수 계산을 위해 false 할당.
			visited[next] = false;
			//마지막 원소의 값 삭제.
			path.remove(here);
		}
		return ret;
	}
}

출력예:
도시의 수?(2<=n<=10):5
[[2, 6, 10, 4, 3], [3, 2, 7, 9, 2], [6, 10, 2, 10, 6], [5, 3, 9, 9, 6], [6, 5, 1, 5, 5]]
17
```
위 코드의 순서를 간단하게 정리하면,
##### 1. 경로를 찾을 기준점(0번 째)도시를 정한다.
##### 2. 모든 도시를 방문하며 이미 방문한 도시인지 확인하고, 경로를 저장한다.
##### 3. 기존의 경로의 길이와 최근에 계산한 경로를 비교하여 더 짧은 경로를 취한다.
##### 4. 가장 짧은 경로를 확정할 때까지 3번을 반복한다.

### 시간 복잡도 분석.
위에서도 잠시 언급했지만, 사실 외판원 문제는 조합을 구하는 문제이므로 위와 같이 완전탐색으로 경우의 수를 계산하면 기준점 도시를 제외하고 도시가 10개라면 경로를 찾아내는데 9!(362,880개)의 경우에 수를 모두 계산해야 하는 난감한? 상황이 연출된다.

여기선 문제의 전제 조건이 n<=10이므로 완전탐색으로도 시간 내에 해결가능하지만, n의 수가 커지면 문제 해결 방법을 동적 계획법이나 다른 방법으로 위 문제를 해결해야 할 것으로 보인다. 그 부분은 책에 진도에 맞춰 다뤄 보는 것으로~

외판원 문제(traveling salesperson problem) 또는 순회 외판원 문제는 조합 최적화 문제의 일종이다. 이 문제는 NP-난해에 속하며, 흔히 계산 복잡도 이론에서 해를 구하기 어려운 문제의 대표적인 예로 많이 다룬다고 한다.([위키 피디아 참고](https://ko.wikipedia.org/wiki/%EC%99%B8%ED%8C%90%EC%9B%90_%EB%AC%B8%EC%A0%9C))

역시 알고리즘 문제는 너무 어렵지만 하나하나 배워가는 재미가 쏠쏠한 것 같다. 오늘도 알고리즘의 매력에 감탄하며 다음은 책의 완전탐색 마지막 문제! [시계 맞추기](https://seongjaemoon.github.io/algorithm/2018/03/06/algorithmBoardCover.html)를 포스팅 해보는 걸로~  


* 오타나 잘못된 부분을 지적해주시면 감사히 생각하고 수정토록 하겠습니다 :)
* 참고문헌<br>
구종만, 알고리즘 문제 해결 전략. pp.165~pp.167.
