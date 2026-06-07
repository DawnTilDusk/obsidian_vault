
1. 左右乘可逆矩阵（初等变换，初等分块变换），rank不变

2. 根据上三角分块矩阵的乘法法则，对角线上的分块在乘方时是各自独立乘方的（你可以手推一下平方或三次方，规律很明显）。例如：$$\begin{bmatrix} J_m(0) & C \\ O & A \end{bmatrix}^m = \begin{bmatrix} (J_m(0))^m & * \\ O & A^m \end{bmatrix}$$
3. Jordan块的转置和旋转180等效，即设$$P_{m} = \begin{pmatrix} & & 1 \\ & \cdot^{\cdot^{\cdot}} & \\ 1 & & \end{pmatrix}$$则有$$P_{m}J_{m}(\lambda_{i})P_{m} = J_{m}^T(\lambda_{i})$$
4. 矩阵的行列式等于其特征多项式的常数项乘 $(-1)^n$，直接将 $\lambda = 0$ 代入 $f(\lambda)$ 得到 $\det(-A)$  $$|A| = \prod \lambda_i = (-1)^n \det(-A) = (-1)^n f(0)$$
5. 逆矩阵的 Jordan 标准型，就是把原矩阵所有的特征值“取倒数”，块的个数和阶数原封不动；而对单个 Jordan 块直接求逆，本质上就是对函数 $f(x)=1/x$ 进行“泰勒展开”
	
	如果 $A \sim J_3(2) \oplus J_2(5)$ 
	那么 $A^{-1} \sim J_3(1/2) \oplus J_2(1/5)$
	比如一个 3 阶的 Jordan 块 $J_3(\lambda) = \begin{pmatrix} \lambda & 1 & 0 \\ 0 & \lambda & 1 \\ 0 & 0 & \lambda \end{pmatrix}$，它的逆矩阵直接是泰勒展开的导数值：

$$J_3(\lambda)^{-1} = \begin{pmatrix} \frac{1}{\lambda} & -\frac{1}{\lambda^2} & \frac{1}{\lambda^3} \\ 0 & \frac{1}{\lambda} & -\frac{1}{\lambda^2} \\ 0 & 0 & \frac{1}{\lambda} \end{pmatrix}$$