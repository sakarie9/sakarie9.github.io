---
title: Image Recognition
date: 2018-06-05
categories: Archived
---

[第十一届河南省ACM程序设计竞赛 I题][1]

<!-- more -->

```cpp
#include "stdafx.h"
#include <iostream>
#include <vector>
#include <cstring>
using namespace std;

long long source[101][101];//原始矩阵
long long sourcesum;
long long newm[101][101];//新矩阵
int m, n;
long long a[101], b[101];//a为每行的最大数，b为每列最大数
bool av[101], bv[101];//visited
long long minsum;

//时间判断变量定义
long long totaltime = 0;
int requiretime = 0;

int T;

struct zeroSave {
	int x;
	int y;
};

vector<zeroSave> zs;

//初始化
void init()
{
	memset(source, -1, sizeof(source));
	memset(newm, -1, sizeof(newm));
	memset(a, 0, sizeof(a));
	memset(b, 0, sizeof(b));
	memset(av, 0, sizeof(av));
	memset(bv, 0, sizeof(bv));
	minsum = 0;
	sourcesum = 0;
	zs.clear();
	zs.push_back({ 0 });
}

//矩阵输入
void input()
{
	cin >> totaltime >> requiretime;
	cin >> m >> n;
	for (int i = 1; i <= m; i++)
		for (int j = 1; j <= n; j++)
		{
			int tmp = 0;
			cin >> tmp;
			source[i][j] = tmp;
			sourcesum += tmp;
			if (!tmp)
			{
				newm[i][j] = 0;
			}
		}
}
//每行每列最大值计算
void maxcal()
{
	long long max;
	for (int i = 1; i <= m; i++)
	{
		max = 0;
		for (int j = 1; j <= n; j++)
		{
			if (max < source[i][j])
				max = source[i][j];
		}
		a[i] = max;
	}
	for (int j = 1; j <= n; j++)
	{
		max = 0;
		for (int i = 1; i <= m; i++)
		{
			if (max < source[i][j])
				max = source[i][j];
		}
		b[j] = max;
	}
}

//矩阵构造
void construct()
{
	//从主视图和左视图看，最优矩阵生成
	for (int i = 1; i <= m; i++)
		for (int j = 1; j <= n; j++)
			if (a[i] == b[j] && bv[j] == 0)
			{
				av[i] = 1;
				bv[j] = 1;
				if (source[i][j] != 0)
				{
					newm[i][j] = a[i];
					break;
				}
				else
				{//碰到合并到0的情况先保存，后面处理
					zeroSave temp;
					temp.x = i; temp.y = j;
					zs.push_back(temp);
					break;
				}
			}

	//剩余最大值插入
	//行剩余处理
	for (int i = 1; i <= m; i++)
	{
		if (av[i] == 1)
			continue;
		for (int j = 1; j <= n; j++)
		{
			if (a[i] <= b[j] && source[i][j] != 0)
			{
				newm[i][j] = a[i];
				av[i] = 1;
				break;
			}
		}
	}
	//列剩余处理
	for (int j = 1; j <= n; j++)
	{
		if (bv[j] == 1)
			continue;
		for (int i = 1; i <= m; i++)
		{
			if (b[j] <= a[i] && source[i][j] != 0)
			{
				newm[i][j] = b[j];
				bv[j] = 1;
				break;
			}
		}
	}
}

//对最大值合并到0上时的处理
void zero()
{
	for (unsigned k = 1; k < zs.size(); k++)
	{
		int flag = 0;
		//查询当前0的横方向上是否有跟最大值相等的数
		for (int i = 1; i <= n; i++)
		{
			if (a[zs[k].x] == b[i] && i != zs[k].y)//如果存在
			{
				flag = 1;
				break;
			}
		}
		if (flag == 0)//如果不存在
		{
			//横向分散
			for (int i = 1; i <= n; i++)
				if (newm[zs[k].x][i] == -1)
				{
					newm[zs[k].x][i] = a[zs[k].x];
					break;
				}
			//纵向分散
			for (int i = 1; i <= m; i++)
				if (newm[i][zs[k].y] == -1)
				{
					newm[i][zs[k].y] = b[zs[k].y];
					break;
				}
		}
	}
}

//计算新矩阵大小
void newMatrixSum()
{
	minsum = 0;
	for (int i = 1; i <= m; i++)
		for (int j = 1; j <= n; j++)
			if (newm[i][j] != -1)
				minsum += newm[i][j];
			else
			{
				minsum++;
				newm[i][j] = 1;
			}
}

//oj输出
void oj()
{
	cin >> T;
	long long diff;
	while (T--)
	{
		init();
		input();
		maxcal();
		construct();
		zero();
		newMatrixSum();

		diff = sourcesum - minsum;
		if (totaltime / (double)requiretime > diff)
		{
			cout << diff << endl;
		}
		else
		{
			cout << totaltime / requiretime << endl;
		}
	}
}

int main()
{
	oj();
	system("pause");
}
```


  [1]: http://acm.hi-54.com/problem.php?pid=2113
