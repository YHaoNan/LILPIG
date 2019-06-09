---
title: 矩阵
date: 2019-05-16 10:10:27
tags: [Numpy,数学]
categories: numpy
mathjax: true
---
*注意：本篇中Latex生成的数学表达式可能有问题*

Numpy实际就是一个矩阵运算库，数据分析这些工作实际上也都是在操作矩阵。

矩阵就是个多维数组，m行n列的矩阵称为m*n矩阵。
$A=\left[
    \begin{matrix}
    	1&2&3\\
    	4&5&6\\
    	7&8&9
    \end{matrix}
\right]\\
\\
A_{2,3}=6$

### 矩阵加法

矩阵加法需要满足两个矩阵大小相同，比如。
$
A=\left[\begin{matrix}
	1&2&3\\
	4&5&6\\
	7&8&9
\end{matrix}
\right]
B=\left[    \begin{matrix}        
	1&2&3\\
	4&5&6\\
	7&8&9
\end{matrix}\right]
A+B=\left[\begin{matrix}        
	2&4&6\\
	8&10&12\\
	14&16&18    
\end{matrix}\right]
$

### 矩阵乘法

矩阵的乘法只有在A为`m*k`矩阵，B为`k*n`矩阵时才有定义，也就是说只有第一个矩阵列数等于第二个行数的时候才可以进行乘法运算。
$
AB_{ij}=\sum_{n=1}^kA_{in}B_{nj}
$
A的第i行和B的第j列的元素依次相乘并相加得到的就是AB的第i,j个元素，AB最后为m*n矩阵。

如：
$
A=\left[    \begin{matrix}
	2&0&5\\
	1&5&3
\end{matrix}\right]
B=\left[\begin{matrix}
	2&1\\
	4&4\\
	1&1
\end{matrix}\right]
A×B=\left[\begin{matrix}
	9&7\\
	25&24
\end{matrix}\right]
$

### 单位矩阵

单位矩阵是一个`n*n`的，由0和1组成的矩阵，定义如下：
$
I_n=[δ_{ij}]，\delta_{ij}=1如果i=j，\delta_{ij}=0如果i \neq j\\
I_n=\left[\begin{matrix}
	1&0&\cdots&0\\
	0&1&\cdots&0\\
	\vdots&\vdots&&\vdots\\
	0&0&\cdots&1
\end{matrix}\right]
$
一个矩阵乘以合适的单位矩阵不会改变该矩阵。
$
当A为m×n矩阵时，有\\
AI_n=I_mA=A
$


### 矩阵转置

$
A=\left[\begin{matrix}        
	2&0&5\\
	1&5&3
\end{matrix}\right]
A^T=\left[\begin{matrix}        
	2&1\\
	0&5\\
	5&3
\end{matrix}\right]
$

