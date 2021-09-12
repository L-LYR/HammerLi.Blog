---
Author: HammerLi
Date: 2021.3.23
Tag: [Algorithm][Search]
---

# 搜索

搜索实质上是一种依据既定策略的枚举。

## Brute Force

暴力搜索其实就是枚举，最好要满足以下条件：

- 枚举对象是连续的，能够使用 for 循环来完成。
- 枚举内容已知，即解空间已知，否则无法完整搜索解空间，也就无法获得最优解。

## DFS

深度优先搜索从起始状态开始利用已知规则将解空间建立为搜索树，对该搜索树进行先序遍历。

```pseudocode
void dfs(current state, depth, others) {
	if (depth == max_depth) {
		update answer;
		return;
	}
	prune;
	for (each possible states) {
		update state;
		dfs(new state, depth + 1, others);
		restore state;
	}
}
```

## BFS

广度优先搜索同理，只不过是对搜索树进行层序遍历。

```pseudocode
void bfs(start state, target) {
	initialize queue;
	enqueue start state;
	while (queue is not empty) {
		dequeue the first state in queue;
		if (current state == target) {
			break;
		}
		for (each possible states) {
			enqueue unvisited state;
		}
	}
}
```

