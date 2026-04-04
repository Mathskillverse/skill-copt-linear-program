# 测试用例集

用于测试 `solve_lp` 函数的各种场景。

## 测试 1：基础最大化（两产品生产）

```python
from solve_lp import solve_lp

result = solve_lp(
    c=[3, 5],
    A_ub=[[1, 2], [2, 1]],
    b_ub=[100, 120],
    bounds=[(0, None), (0, None)],
    sense="max",
)
# 预期：status="OPTIMAL", obj≈293.33, x≈[40, 30]
print(result)
```

**题意**：产品 A 利润 3 元，产品 B 利润 5 元。原料约束：A 用 1 单位、B 用 2 单位，共 100。工时约束：A 用 2 小时、B 用 1 小时，共 120 小时。

---

## 测试 2：基础最小化（饮食问题）

```python
result = solve_lp(
    c=[9, 7],
    A_ub=[[-3, -2], [-1, -2]],  # >= 约束转为 <=
    b_ub=[-100, -80],
    bounds=[(0, None), (0, None)],
    sense="min",
)
# 预期：status="OPTIMAL", obj≈306.67, x≈[13.33, 33.33]
print(result)
```

**题意**：原料 A 成本 9 元，原料 B 成本 7 元。营养 P 至少 100（A 提供 3、B 提供 2），营养 Q 至少 80（A 提供 1、B 提供 2）。

---

## 测试 3：运输问题

```python
result = solve_lp(
    c=[4, 6, 5, 3],  # W1->S1, W1->S2, W2->S1, W2->S2
    A_ub=[
        [1, 1, 0, 0],    # W1 供给 <= 80
        [0, 0, 1, 1],    # W2 供给 <= 70
        [-1, 0, -1, 0],  # S1 需求 >= 60 (转 <= -60)
        [0, -1, 0, -1],  # S2 需求 >= 50 (转 <= -50)
    ],
    b_ub=[80, 70, -60, -50],
    bounds=[(0, None)] * 4,
    sense="min",
)
# 预期：status="OPTIMAL", obj≈740, x=[60, 20, 0, 50]
print(result)
```

**题意**：两仓库两门店，求最小运费。

---

## 测试 4：含变量上界

```python
result = solve_lp(
    c=[8, 11],
    A_ub=[[2, 3], [1, 1]],
    b_ub=[120, 90],
    bounds=[(0, 30), (0, 25)],  # 市场销量上限
    sense="max",
)
# 预期：status="OPTIMAL", obj≈390, x=[30, 25]
print(result)
```

---

## 测试 5：仅等式约束

```python
result = solve_lp(
    c=[5, 8, 2],
    A_eq=[[1, 1, 1], [1, 0, 0]],
    b_eq=[10, 3],
    bounds=[(0, None), (0, None), (0, None)],
    sense="min",
)
# 预期：status="OPTIMAL", obj=29, x=[3, 5, 2]
print(result)
```

---

## 测试 6：无可行解（INFEASIBLE）

```python
result = solve_lp(
    c=[1, 1],
    A_ub=[[1, 1], [-1, -1]],
    b_ub=[10, -20],  # x1+x2<=10 且 x1+x2>=20，矛盾
    bounds=[(0, None), (0, None)],
    sense="min",
)
# 预期：status="INFEASIBLE", obj=None, x=None
print(result)
```

---

## 测试 7：无界解（UNBOUNDED）

```python
result = solve_lp(
    c=[-1, -1],  # 最小化 -(x1+x2)，相当于最大化 x1+x2
    bounds=[(0, None), (0, None)],
    sense="min",
)
# 预期：status="UNBOUNDED", obj=None, x=None
print(result)
```

**题意**：没有约束限制变量增长，目标可无限减小。

---

## 测试 8：维度错误（应抛出异常）

```python
try:
    result = solve_lp(
        c=[1, 2, 3],
        A_ub=[[1, 2]],  # 只有 2 列，但 c 有 3 个元素
        b_ub=[10],
        sense="min",
    )
except ValueError as e:
    print(f"捕获预期错误：{e}")
```

---

## 测试 9：完整产销平衡问题

```python
result = solve_lp(
    c=[2, 3, 1, 4, 5, 2],  # 3 个工厂到 2 个市场
    A_ub=[
        [1, 1, 0, 0, 0, 0],  # 工厂 1 产能 <= 50
        [0, 0, 1, 1, 0, 0],  # 工厂 2 产能 <= 60
        [0, 0, 0, 0, 1, 1],  # 工厂 3 产能 <= 40
        [-1, 0, -1, 0, -1, 0],  # 市场 1 需求 >= 70
        [0, -1, 0, -1, 0, -1],  # 市场 2 需求 >= 60
    ],
    b_ub=[50, 60, 40, -70, -60],
    bounds=[(0, None)] * 6,
    sense="min",
)
# 预期：status="OPTIMAL", obj≈380
print(result)
```

---

## 测试 10：资源配置（3 变量）

```python
result = solve_lp(
    c=[50, 80, 40],  # 三种产品利润
    A_ub=[
        [2, 3, 1],   # 原料 A 总量 <= 200
        [1, 2, 3],   # 原料 B 总量 <= 180
        [4, 2, 1],   # 工时总量 <= 300
    ],
    b_ub=[200, 180, 300],
    bounds=[(0, None), (0, None), (0, None)],
    sense="max",
)
# 预期：status="OPTIMAL"
print(result)
```

---

## 批量测试脚本

```python
if __name__ == "__main__":
    from solve_lp import solve_lp

    tests = [
        ("基础最大化", {
            "c": [3, 5], "A_ub": [[1, 2], [2, 1]], "b_ub": [100, 120],
            "bounds": [(0, None), (0, None)], "sense": "max",
        }),
        ("基础最小化", {
            "c": [9, 7], "A_ub": [[-3, -2], [-1, -2]], "b_ub": [-100, -80],
            "bounds": [(0, None), (0, None)], "sense": "min",
        }),
        ("无可行解", {
            "c": [1, 1], "A_ub": [[1, 1], [-1, -1]], "b_ub": [10, -20],
            "bounds": [(0, None), (0, None)], "sense": "min",
        }),
        ("无界解", {
            "c": [-1, -1], "bounds": [(0, None), (0, None)], "sense": "min",
        }),
    ]

    for name, params in tests:
        try:
            r = solve_lp(**params)
            print(f"{name}: {r['status']}, obj={r['obj']}")
        except Exception as e:
            print(f"{name}: 错误 - {e}")
```