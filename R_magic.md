`%%R`：

```
%R [-i INPUT] [-o OUTPUT] [-n] [-w WIDTH] [-h HEIGHT] [-p POINTSIZE]
     [-b BG] [--noisolation] [-u {px,in,cm,mm}] [-r RES]
     [--type {cairo,cairo-png,Xlib,quartz}] [-c CONVERTER] [-d DISPLAY]
     [code ...]
```

在 R 中执行代码，并可选择将结果返回给 Python 运行时。

在行模式下，这将评估一个表达式并将返回的值转换为 Python 对象。返回值由 `rpy2` 的行为决定，即返回评估最后一个表达式的结果。

多个 R 表达式可以通过分号连接在一起执行：

```python
In [9]: %R X=c(1,4,5,7); sd(X); mean(X)
Out[9]: array([ 4.25])
```

在单元格模式下，这将运行一段 R 代码。如果在标准的 R REPL 环境中评估相同的代码，结果会被打印出来。

在单元格模式下，默认情况下不会将任何结果返回给 Python：

```python
In [10]: %%R
   ....: Y = c(2,4,3,9)
   ....: summary(lm(Y~X))
```

输出：

```
Call:
lm(formula = Y ~ X)

Residuals:
    1     2     3     4
 0.88 -0.24 -2.28  1.64

Coefficients:
              Estimate Std. Error t value Pr(>|t|)
(Intercept)   0.0800     2.3000   0.035    0.975
X             1.0400     0.4822   2.157    0.164

Residual standard error: 2.088 on 2 degrees of freedom
Multiple R-squared: 0.6993, Adjusted R-squared: 0.549
F-statistic: 4.651 on 1 and 2 DF, p-value: 0.1638
```

在笔记本中，图表作为单元格输出显示：

```python
%R plot(X, Y)
```

这将创建一个 X 和 Y 的散点图。

如果 `cell` 不为 `None` 且行中有 R 代码，它会被添加到单元格中的 R 代码之前。

对象可以通过 `-i` 和 `-o` 标志在 `rpy2` 和 Python 之间传递：

```python
In [14]: Z = np.array([1,4,5,10])

In [15]: %R -i Z mean(Z)
Out[15]: array([ 5.])

In [16]: %R -o W W=Z*mean(Z)
Out[16]: array([  5.,  20.,  25.,  50.])

In [17]: W
Out[17]: array([  5.,  20.,  25.,  50.])
```

返回值由以下规则决定：

- 如果 `cell` 不为 `None`（即有内容），魔法命令返回 `None`。
- 如果评估最后一行时返回 `NULL`，则返回 `None`。
- 不会尝试将最后的值转换为结构化数组。使用 `%Rget` 将结构化数组推送到 Python。
- 如果存在 `-n` 标志，则不返回任何值。
- 行末的分号也会导致没有返回值，因为行末的最后一个值为空字符串。

选项：

- `-i INPUT, --input INPUT`：Python 对象的名称，这些对象将在使用默认转换器或通过 `-c/--converter` 指定的转换器后分配给 R 对象。多个输入可以用逗号分隔，不能有空格。例如，`-i myvariable` 或 `-i mypackage.myothervariable` 都可以工作。每个输入可以是 Python 对象的名称，也可以是类似 `<r-name>=<python-name>` 的表达式。
- `-o OUTPUT, --output OUTPUT`：执行单元格体后，要从 `rpy2` 推送到 `shell.user_ns` 的变量名（`rpy2` 内部会根据需要应用 `ri2ro`）。多个名称用逗号分隔。
- `-n, --noreturn`：强制魔法命令不返回任何值。

绘图：

- `-w WIDTH, --width WIDTH`：R 中绘图设备的宽度。
- `-h HEIGHT, --height HEIGHT`：R 中绘图设备的高度。
- `-p POINTSIZE, --pointsize POINTSIZE`：R 中绘图设备的点大小。
- `-b BG, --bg BG`：R 中绘图设备的背景色。

SVG：

- `--noisolation`：禁用笔记本中的 SVG 隔离。默认情况下，SVG 会被隔离，以避免图形之间的命名空间冲突。禁用 SVG 隔离后，可以引用先前的图形或在多个 SVG 之间共享 CSS 规则。

PNG：

- `-u <{px,in,cm,mm}>, --units <{px,in,cm,mm}>`：发送到 R 中 *png* 绘图设备的单位，可以是 `["px", "in", "cm", "mm"]`。
- `-r RES, --res RES`：发送到 R 中 *png* 绘图设备的分辨率。如果 *units* 为 `["in", "cm", "mm"]`，默认分辨率为 72。
- `--type <{cairo,cairo-png,Xlib,quartz}>`：用于生成图形的设备类型。
- `-c CONVERTER, --converter CONVERTER`：用于转换的本地转换器的名称。转换器包含在 Python 和 R 之间转换对象的规则。如果未指定或为 `None`，则使用魔法命令模块的默认转换器（即 `rpy2` 的默认转换器 + numpy 转换器 + pandas 转换器，如果都可用）。
- `-d DISPLAY, --display DISPLAY`：用于显示 R 单元格输出的函数名称（该输出是没有左侧赋值的最后一个对象或函数调用）。该函数的签名为 `(robject, args)`，其中 `robject` 是单元格的 R 输出对象，`args` 是传递给单元格的所有参数的命名空间。

--- 

- **%R** 用于执行单行 R 代码并返回结果。
- **%%R** 用于执行一个 R 代码块，并将其结果打印出来。
- 可以通过 `-i` 和 `-o` 参数在 Python 和 R 之间传递数据。
- 还有一些绘图和图像处理相关的参数，如 `-w`, `-h`, `-r` 等。