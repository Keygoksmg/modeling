# 例


```python
import pulp
from itertools import product

problem = pulp.LpProblem('Problem_1', pulp.LpMaximize)

# 変数の定義
'''実数ならContinuous, 整数はInteger'''
x = pulp.LpVariable('x', lowBound=0, cat='Continuous')
y = pulp.LpVariable('y', lowBound=0, cat='Continuous')

# 目的関数
problem += 120*x + 200*y

# 制約条件を設定
problem += x + 3*y <= 16
problem += 4*x + y <= 18

# 実行
status = problem.solve()
print('Status', pulp.LpStatus[status])

# 結果の表示
print('Result')
print('x =', x.value())
print('y =', y.value())
print('目的関数値 =', problem.objective.value())

```

    Writing kadai1.tex


# 課題１｜仕事の割当問題


```python
import pulp
import time

staff_num = 5 
job_num = 3

# 変更前
# required = [2,2,1] 
# cost=[[5,4,2,3,1], [4,5,1,2,1], [3,3,4,2,2]]

# # 変更後１
# required = [2,2,1] 
# cost=[[6,5,3,4,2], [5,6,2,3,2], [4,4,5,3,3]]

# 変更後２
required = [2,1,2] 
cost=[[6,5,3,4,2], [5,6,2,3,2], [4,4,5,3,3]]

assignment = pulp.LpProblem("example", pulp.LpMinimize) # pulp.LpMinimize : 最小化
# pulp.LpMaximize : 最大化

# 変数宣言
'''# lowBound, upBound を指定しないと、それぞれ -無限大, +無限大 になる 
# pulp.LpContinuous : 連続変数
# pulp.LpInteger : 整数変数
# pulp.LpBinary : 0-1変数
'''
x=[[0 for i in range(staff_num)]for j in range(job_num)]
print(x)
for j in range(job_num):
    for i in range(staff_num):
        x[j][i] = pulp.LpVariable(f"x({j},{i})", 0, 1, pulp.LpInteger)

# 目的関数
assignment += sum(cost[j][i] * x[j][i] for i in range(staff_num) for j in range(job_num))

# 制約条件
for i in range(staff_num):
    assignment += sum(x[j][i] for j in range(job_num)) == 1 
for j in range(job_num):
    assignment += sum(x[j][i] for i in range(staff_num)) >= required[j]

# 時間計測開始
time_start = time.perf_counter()
status = assignment.solve() 

# 時間計測終了
time_stop = time.perf_counter()

print(pulp.LpStatus[status]) 
for i in range(staff_num):
    for j in range(job_num):
        if x[j][i].value() == 1 :
            print(i,"->",j)

print("Obj.value", assignment.objective.value())
print("time:", time_stop-time_start)
```

    [[0, 0, 0, 0, 0], [0, 0, 0, 0, 0], [0, 0, 0, 0, 0]]
    Optimal
    0 -> 2
    1 -> 2
    2 -> 1
    3 -> 0
    4 -> 0
    Obj.value 16.0
    time: 0.026043491999985235


# 課題２｜ナンバープレース


```python
import pulp
from itertools import product
from pprint import pprint

matrix1 = [[0, 4, 0, 0, 0, 0, 0, 0, 5],
         [0, 0, 0, 0, 0, 0, 0, 8, 0],
         [1, 0, 0, 0, 0, 0, 0, 4, 6],
         [8, 0, 3, 0, 1, 9, 0, 0, 0],
         [6, 0, 0, 0, 0, 8, 2, 0, 0],
         [0, 0, 0, 0, 4, 3, 0, 0, 0],
         [3, 0, 0, 0, 0, 0, 0, 0, 1],
         [0, 0, 0, 5, 8, 1, 9, 0, 0],
         [4, 0, 8, 0, 0, 0, 0, 0, 0]] 

matrix2 = [[6, 0, 2, 0, 0, 0, 0, 0, 0],
         [0, 0, 8, 0, 2, 7, 0, 0, 3],
         [0, 0, 0, 0, 0, 0, 0, 0, 0],
         [0, 0, 0, 0, 0, 0, 0, 1, 0],
         [4, 5, 0, 0, 3, 0, 0, 0, 0],
         [1, 3, 0, 0, 7, 0, 0, 6, 2],
         [0, 1, 0, 0, 6, 3, 0, 9, 0],
         [0, 0, 0, 0, 0, 1, 0, 0, 0],
         [2, 0, 5, 0, 0, 0, 8, 0, 0]]

def number_place(matrix):
    # Object
    problem = pulp.LpProblem()

    # 問題｜０は空白
    matrix = matrix

    # 定数
    LINE = list(range(9))

    # 変数宣言
    x = [[[pulp.LpVariable(f'x{i}{j}{k}', cat='Binary')
           for k in LINE] for j in LINE] for i in LINE]           

    # 目的関数

    # 制約条件
    '''
    条件１：一つのマスに一つの数字
    条件２：縦列jにその数字がちょうど一つある
    条件３：縦列iにその数字がちょうど一つある
    条件４：各3*3のブロック内に一つの数字がある
    '''
    # すでに数字があるセルを１に変更
    for i, j in product(LINE, LINE):
        if matrix[i][j] > 0:
            problem += x[i][j][matrix[i][j]-1] == 1

    # 上記の条件
    for i, j in product(LINE, LINE):
        problem += pulp.lpSum(x[i][j][k] for k in LINE) == 1

    for i, k in product(LINE, LINE):
        problem += pulp.lpSum(x[i][j][k] for j in LINE) == 1

    for j, k in product(LINE, LINE):
        problem += pulp.lpSum(x[i][j][k] for i in LINE) == 1

    block = [[b, b+1, b+2] for b in [0, 3, 6]]
    for a, b in product(block, block):
        for k in LINE:
            problem += pulp.lpSum(x[i][j][k] for i in a for j in b) == 1

    # 時間計測
    time_start = time.perf_counter()
    status = problem.solve() # 実行
    time_stop = time.perf_counter()

    # 結果
    print('Result')
    for i, j, k in product(LINE, LINE, LINE):
        if x[i][j][k].value() == 1:
            matrix[i][j] = k + 1
        
    pprint(matrix)
    print("Time:", time_stop-time_start)
    
number_place(matrix1)
number_place(matrix2)
```

    Result
    [[9, 4, 6, 8, 3, 7, 1, 2, 5],
     [2, 3, 5, 1, 6, 4, 7, 8, 9],
     [1, 8, 7, 9, 2, 5, 3, 4, 6],
     [8, 2, 3, 6, 1, 9, 4, 5, 7],
     [6, 9, 4, 7, 5, 8, 2, 1, 3],
     [5, 7, 1, 2, 4, 3, 6, 9, 8],
     [3, 5, 9, 4, 7, 2, 8, 6, 1],
     [7, 6, 2, 5, 8, 1, 9, 3, 4],
     [4, 1, 8, 3, 9, 6, 5, 7, 2]]
    Time: 0.06450084500102093
    Result
    [[6, 7, 2, 3, 1, 4, 9, 5, 8],
     [5, 9, 8, 6, 2, 7, 1, 4, 3],
     [3, 4, 1, 9, 8, 5, 6, 2, 7],
     [8, 2, 7, 5, 9, 6, 3, 1, 4],
     [4, 5, 6, 1, 3, 2, 7, 8, 9],
     [1, 3, 9, 4, 7, 8, 5, 6, 2],
     [7, 1, 4, 8, 6, 3, 2, 9, 5],
     [9, 8, 3, 2, 5, 1, 4, 7, 6],
     [2, 6, 5, 7, 4, 9, 8, 3, 1]]
    Time: 0.036229927000022144



```python

```
