BFS (너비 우선 탐색)
==================================

BFS는 탐색을 할 때 너비를 우선으로 수행하는 그래프 탐색 알고리즘이다.
너비 우선 탐색은 최단 경로를 찾아야 할 때 많이 사용한다.
재귀 함수로 구현하지 않고 큐(Queue)로 주로 구현한다.

C++ BFS 구현
```C++
bool visited[101] = {};
void bfs(vector<int> graph[], int start) {
	queue<int> q;
	q.push(start);
	visited[start] = true;

	while (!q.empty()) {
		int x = q.front();
		q.pop();
		cout << x << " ";
		for (int i = 0; i < graph[x].size(); i++) {
			int y = graph[x][i];
			if (!visited[y]) {
				q.push(y);
				visited[y] = true;
			}
		}
	}
}
```

