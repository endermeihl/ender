在LaTeX中输入数学符号时，通常使用 `$...$` 或 `$$...$$` 来表示数学环境。其中 `$ ... $` 表示行内公式，`$$ ... $$` 表示独立的行间公式。以下是一些常用数学符号的输入方法：

### 1. 基础运算符

- 加法：`+`
- 减法：`-`
- 乘法：`\times`（或`\cdot`用于点乘）$\times$
- 除法：`\div` 或 `\frac{a}{b}`（分数）
- 等于：`=`

### 2. 括号和分隔符

- 圆括号：`( ... )`
- 方括号：`[ ... ]`
- 大括号：`\{ ... \}`
- 绝对值：`\| ... \|`
- 大的括号：`\left( ... \right)`

### 3. 指数和下标

- 指数：`a^b`
- 下标：`a_b`

### 4. 根号

- 平方根：`\sqrt{x}`
- n次根：`\sqrt[n]{x}`

### 5. 求和与积分

- 求和符号：`\sum_{i=1}^n`
- 积分符号：`\int_{a}^{b}`
- 重积分：`\iint`，三重积分 `\iiint`

### 6. 极限与导数

- 极限：`\lim_{x \to \infty}` $\lim_{x \to \infty}$
- 微分：`\frac{dy}{dx}` $`\frac{dy}{dx}`$
- 偏导数：`\frac{\partial y}{\partial x}` $\frac{\partial y}{\partial x}$

### 7. 常用希腊字母

- α：`\alpha`
- β：`\beta`
- γ：`\gamma`
- Δ：`\Delta`
- π：`\pi`
- Σ：`\Sigma`

### 8. 逻辑符号

- 小于：`<`，大于：`>`
- 小于等于：`\leq` $\leq$，大于等于：`\geq` $\geq$
- 不等于：`\neq` $\neq$
- 并集：`\cup` $\cup$
- 交集：`\cap` $\cap$
- 属于：`\in` $\in$
- 不属于：`\notin` $\not\in$

### 9. 箭头和向量

- 向量箭头：`\vec{v}` $\vec{v}$
- 向左箭头：`\leftarrow` $\leftarrow$
- 向右箭头：`\rightarrow` $\rightarrow$
- 双向箭头：`\leftrightarrow` $\leftrightarrow$

### 10. 花体字母与其他符号

- 花体字母：`\mathcal{A}` $\mathcal{A}$
- 空集：`\emptyset` $\{  \}$
- 无穷大：`\infty` $\infty$

### 示例

假设你想输入一个公式：

$$
f(x) = \sum_{i=1}^{n} x_i^2 + \int_{a}^{b} x \, dx
$$

对应的LaTeX代码为：

```latex
f(x) = \sum_{i=1}^{n} x_i^2 + \int_{a}^{b} x \, dx
```

这些是LaTeX中常用的数学符号，可以帮助你表达数学公式。