title: dfs
tags:
	- 拓扑排序
	- dfs
categories: algorithm
---

深度优先搜索算法一般用于树或者图的结构中，在写代码时有下面几个注意的地方。

leetcode 207题
首先读懂题目，课程只有0~crousenum-1门，如果没有课程相互前置，是一定能够学完的。题目就是判断是否有循环依赖。可以使用广度+深度搜索，遍历一个节点所依赖节点，如果所依赖节点也有依赖则继续判断。

这里每个节点在遍历时赋予了属性：未检索、正在检索、已经完成检索。为什么不是设置未检索和已经完成检索两个值。主要原因是需要判断是否有相互依赖。设置已经完成检索需要在完成该节点的依赖遍历后，但遍历的节点如果依赖正在的遍历的节点，那只设置两个值的话，就会检测不出来。

```go
canFinish(numCourses int, prerequisites [][]int) bool {
    depend := make([][]int, numCourses)
	flag := make([]int, numCourses)
	for i := 0; i < len(prerequisites); i++ {
		depend[prerequisites[i][0]] = append(depend[prerequisites[i][0]], prerequisites[i][1])
	}
	var dfs func(int) bool
	dfs = func(n int) bool {
		if flag[n] == 1 {
			return false
		}
		flag[n] = 1
		for i := 0; i < len(depend[n]); i++ {
			if flag[depend[n][i]] == 1 {
				return false
			} else if flag[depend[n][i]] == 0 {
				if !dfs(depend[n][i]) {
					return false
				}
			}
		}
		flag[n] = 2
		return true
	}
	for i := 0; i < numCourses; i++ {
		if flag[i] != 2 && !dfs(i) {
			return false
		}
	}
	return true
}
```
