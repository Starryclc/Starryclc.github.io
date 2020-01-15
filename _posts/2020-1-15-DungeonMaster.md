---
layout: post
title: DungeonMaster
categories: [学习笔记,算法,简单搜索]
---

## 问题描述
3D迷宫问题，找到耗时最短路径。所以用bfs来搜索。

## 解题代码

```c++
#include<cstdio>
#include<cstring>
#include<iostream>
#include<queue>
using namespace std;

struct node{
	int l, r, c;
	int time;
};
node start, End;
queue <node> que;
int dir[6][3] = { {0,0,1},{0,0,-1},{0,1,0},{0,-1,0},{1,0,0},{-1,0,0} };
char cube[30][30][30];
int vis[30][30][30];
int L, R, C;

int bfs()
{
	que.push(start);
	vis[start.l][start.r][start.c] = 1;
	while (!que.empty()) {
		node front = que.front();
		que.pop();
		if (front.l == End.l && front.r == End.r && front.c == End.c) {
			return front.time;
		}
		for (int i = 0; i < 6; i++) {
				node tmp;
				tmp.l = front.l + dir[i][0];
				tmp.r = front.r + dir[i][1];
				tmp.c = front.c + dir[i][2];
				if (tmp.l >= 0 && tmp.l < L && tmp.r >=0 &&tmp.r<R&&tmp.c<C&& tmp.c >=0 
					&& cube[tmp.l][tmp.r][tmp.c] != '#'&& vis[tmp.l][tmp.r][tmp.c] ==0) {
					//printf("%d %d %d\n", tmp.l,tmp.r,tmp.c);
					vis[tmp.l][tmp.r][tmp.c] = 1;
					tmp.time=front.time+1;
					que.push(tmp);
				}
			}
			
		}
	return -1;
	}

int main(void) 
{
	while (cin >> L >> R >> C)
	{
		
		if (L == 0 && R == 0 && C == 0) break;
		memset(vis, 0, sizeof(vis));
		while (!que.empty()) {
			que.pop();
		}
		for (int i = 0; i < L; i++) {
			for (int j = 0; j < R; j++) {
				cin >> cube[i][j];
				for (int k = 0; k < C; k++) {
					if (cube[i][j][k] == 'S') {
						start.l = i; start.r = j; start.c = k; start.time = 0;
					}
					else if(cube[i][j][k]=='E'){
						End.l = i; End.r = j; End.c = k; End.time = 0;
					}
				}
			}
		}
		int TIME = bfs();
		if (TIME == -1) printf("Trapped!\n");
		else printf("Escaped in %d minute(s).\n", TIME);
	}
}
```

## 总结

* Queue
  队列先进先出的特点正好符合树中的广度优先。
* dis数组控制六个方向
* node结构体表示每个节点，包含自己的坐标和到达所用时间

* 一开始没A，要注意每组处理结束后清空队列：因为找到最短路径的时候，可能队列后面还有很多节点，所以在进行下一组数据处理之前要把队列里可能的节点清空。
