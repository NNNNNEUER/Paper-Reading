# CrypTFlow2: Practical 2-Party Secure Inference

### Part 0 Background and Motivation

    The prediction of neural network inference under 2PC is always a difficult research problem. The first difficulty is the huge cost of nonlinear computation, such as comparison and division. The other is the loss of accuracy.

    CrypTFlow2 proposes a new protocol for security comparison based on Oblivious Transfer (OT) and deeply optimizes the protocol. Furthermore, several operator protocols for neural networks are designed based on the comparison protocol, such as ReLU, Truncation, faithful Division (divisor is public), Avgpool, and Maxpool.

    In addition, two versions of the ring$\ \mathbb{Z}_L(L = 2^l)$ and $\mathbb{Z}_n$ (n is any large integer) are designed in this paper, so as to adapt the linear layer computation for OT and Homomorphic Encryption (HE).

    Finally, the code for this article has been open sourced, linked to:[https://github.com/mpc-msri/EzPC/tree/master/SCI]()

### Part 1 Premliminaries

    This paper is aimed at the semi-honest adversaries in two-party computation, mainly using the additive secret sharing under two parties, OT, OT constructed AND triples, Multiplexer, and B2A transformation, and HE, where secret sharing is already familiar, and HE is mainly used in the linear layer of neural networks. The core construction of this article is based on OT, and we mainly review the knowledge of OT here.

*Notation :*

* $$
  w\leftarrow^{s} W
  $$

  For a set W, w is an element randomly selected from W.
* $$
  [l]
  $$

  denotes the set of integers ${0,..,l-1}$
* $$
  1\{b\}
  $$

  denote the indicator function that is 1 when ? is true and 0 when ? is false.

##### 1.1 Oblivious Transfer

    OT is very clear in terms of its function. The standard definition of 1-out-of-2 OT involves two parties: the sender S holds two bits of information, and the receiver R holds a selection bit. After the OT protocol is executed,R can get the information of the index corresponding to the selection bit, but cannot get other information, and S does not know which information R selects. It can be generalized to 1-out-of-n OT, that is, the receiver needs to secretly obtain a certain information from the n messages of the sender. Similarly, it is further extended to k-out-of-n OT, that is, out of n messages from the sender, the receiver secretly obtains k messages.

![1730708346728](https://file+.vscode-resource.vscode-cdn.net/h%3A/%E5%A4%87%E4%BB%BDG%E7%9B%98/EDA/fundamentals/image/OT/1730708346728.png)

    In the OT protocol, the sender has all the data rights, the receiver has the option of a single data, the data exchange is completed inadvertently, while ensuring the privacy of the private data of both sides.

##### 1.2 OT-based MUX and B2A

###### $F_{MUX}$:

* input: $\lang a\rang^n$ and $\lang c\rang^B$
* output: if c = 1, output $\lang a\rang^n$; otherwise, output shares over 0

Based on $\begin{pmatrix}2\\1\end{pmatrix}-OT_{\eta}$, $x_0 = -r_0 + c\cdot\lang a\rang^n_0$, $\ x_1 = -r_1 + c\cdot\lang a\rang^n_1$. Therefore, $z = z_0 + z_1 = c\cdot a$. Since OT is used twice, the communication expense is $2(\lambda + 2\eta)$.

![1730787098708](image/ctf2/1730787098708.png)

###### $F_{B2A}$:

* Input: $\lang c\rang^B$;
* output: $\lang d\rang^n$, which satisfies $d=c$.

Since $d = \lang c\rang^B_0 + \lang c\rang^B_1 - 2\lang c\rang^B_0 \cdot \lang c\rang^B_1$, the key is on the product term. Based on $\begin{pmatrix}2\\1\end{pmatrix}-OT_{\eta}$, $y_1 = x+\lang c\rang^B_0 \cdot \lang c\rang^B_1$. Therefore, $\lang d\rang^n_0 = \lang c\rang^B_0  + 2x$. Since OT is used twice, the communication expense is $\lambda + \eta$.

![1730787754171](image/ctf2/1730787754171.png)

### Part 3 Millionaries' and DReLu Protocols (non-linear)

##### Millionaires' protocols

###### General idea

    Input :$ P_0$ has $x\in \{0,1\}^l$ and $ P_1$ has $y\in \{0,1\}^l$

    output :$P_0, P_1$ learn shares of $\{x>y\}$

for $x = x_1||x_0,\ \ y = y_1||y_0$ :

$$
\{x>y\} = \{x_1<y_1\}\oplus (\{x_1 = y_1\}\wedge\{x_0<y_0\})
$$

$$
\{x=y\} = (\{x_1 = y_1\}\wedge\{x_0=y_0\})
$$

> In GSV07, we conduct this comparison until the single bit-leaves

In CRTF2, however, we stop at $q = l/m$ leaves; $x_i,y_i\in\{0,1\}^m$ and conduct$2\times\begin{pmatrix} M\\ 1\end{pmatrix}-OT_1$  on each leaf; $M = 2^m$

    This change is efficient only for small bitlengths, such as$\ m=4$

Communications for $m = 4$: $6\lambda l\rightarrow2\lambda l$

After more optimizations: $2\lambda l\rightarrow\lambda l$

![1730120816252](image/ctf2/1730120816252.png)

###### Protocol 

    As shown in the picture below. Let$\ M = 2^m$ and consider a simple case $q = l/m$, where q is to the power of 2. By recursing $log_2^q$ times, we obtain a tree with q leaves. Each leaf has m bits, i.e. $x = x_{q-1}||...||x_0$ and $y = y_{q-1}||...||y_0$, where $x_i,y_i\in\{0,1\}^m$. By doing so, both parties can take advantage of $\begin{pmatrix}M\\1\end{pmatrix}-OT$ and use its comparision function and equivalence function. After calculating those leaves, we can calculate AND and XOR recursively to the root node in order to get the final output.

![1730122971317](https://file+.vscode-resource.vscode-cdn.net/h%3A/%E5%A4%87%E4%BB%BDG%E7%9B%98/EDA/ctf2/image/ctf2/1730122971317.png)

��ϸ��Э�����£��������Э��ľ������ݣ�
���룺$P_0,\ P_1$ �ֱ�����x,y,������������lλ�Ķ����ƴ�

�����$P_0,\ P_1$ �ֱ�õ��ȽϽ�������ܷ���ֵ��һ������λ��

��һ����$P_0$������x�ֳ�q���Ӵ���$P_1$������y�ֳ�q���Ӵ���ÿ���Ӵ��Ĵ�С��m��

�ڶ�������M=2^m
������������j=0��q-1����ѭ����
���Ĳ���$P_0$���ѡ����������λ������������λ�����ڵõ���������ܷ���ġ�

���岽������k=0��M-1ѭ��.
��������$P_0$���÷��͵�$s_{j,k}$ ��s���ڵõ���ʽ$x_j < k$��ֵ

���߲���$P_0$���÷��͵�$t_{j,k}$ ��t���ڵõ���ʽ$x_j = k$��ֵ

�ڰ˲�������k��ѭ��
�ھŲ���$P_0,\ P_1$����1-out-of-M OTЭ�飬����M������$s_{j,k}$��ѡ��һ����$P_0$��Ϊ���ͷ���������M������$s_{j,k}$��$P_1$��Ϊ���շ���������$y_j$

��ʮ����$P_0,\ P_1$����1-out-of-M OTЭ�飬����M������$t_{j,k}$��ѡ��һ����$P_0$��Ϊ���ͷ���������M������$t_{j,k}$��$P_1$��Ϊ���շ���������$y_j$

��ʮһ��������j��ѭ��
��ʮ����������i����1��l $log_2q$
��ʮ����������j����0��$\frac{q}{2^i}-1$
��ʮ�Ĳ��������ʽ�Ͳ���ʽ����
��ʮ�岽�����ؽ����������Ϳ����ˡ���Ϊ����൱����ӣ��������ܷ���Ŀɼ��ԣ����Ա��ؼ��㡣
��ʮ�����������ʽ�͵�ʽ���룬������һ���ʽ���жϡ�ע���ʱ����λ�ĺ��������ǱȽϵĽ��������x��y�����λ��
��ʮ�߲�������j��ѭ��
��ʮ�˲�������i��ѭ��
��ʮ�Ų���������Ľ����

*See the appendix for a detailed code-reading*

###### Security Proof

The author simply deduced that the resulting output could equal the result of the comparison. Security follows OT

and $F_{AND}$'s Security of hybrid protocols

###### Generalization

1. In the case that m does not divide l, for l mod m=r, which is the remainder of l divided by m, run it separately
2. If q is not a power of 2, consider $2^a < q \leq 2_{a+1}$, So let's take $2^a$ part as a fragment to build a perfect binary tree. Finally, compare the value of the perfect binary tree with the rest of the comparison.

###### Optimization

1. In steps 9 and 10, OT is used to combine two 1-bit $\begin{pmatrix}M\\1\end{pmatrix}-OT$ into a 2-bit $\begin{pmatrix}M\\1\end{pmatrix}-OT$. Because $P_1$'s input is still $y_j$
2. In steps 14 AND 16, we use AND. Use related AND, that is, the AND protocol with the same input, to save expenses.
3. For the entire binary tree, the least significant fork of the equation is never used, so it can be omitted.

###### Communication

![1730788439866](image/ctf2/1730788439866.png)

##### Protocol for DReLU

###### General idea

    Over$\ Z$, $DReLU(x) = \begin{cases}0,\ x<0\\1,\ x\geq0\end{cases}$

    Over$\ Z_n$, $DReLU(x) = \begin{cases}0,\ x\in[\lceil\frac{n}{2}\rceil,n)\\1,\ x\in[0,\lceil\frac{n}{2}\rceil)\end{cases}$

Input: $\ P_0 $ has $x_0 \in [0,n)$ and $\ P_1$ has $x_1\in[0,n)$; s.t. $x_0 + x_1 = x \ mod\ n $

Output: $P_0,\ P_1$ learn shares of $DReLu(x)$

![1730120881231](image/ctf2/1730120881231.png)

Computing DReLU(x):

* Simple: 3 calls to Millionaires'
* Optimized: 2 comparisons
* In particular: for $n = 2^l$, only needs 1 comparison

$$
lt: x_0 + x_1 \geq \lceil\frac{n}{2}\rceil \Leftrightarrow x_0 + \lfloor\frac{n}{2}\rfloor > n-1-x_1
$$

$$
wrap: x_0 + x_1 \geq n \Leftrightarrow x_0  > n-1-x_1
$$

$$
rt: x_0 + x_1 \geq \lceil\frac{3n}{2}\rceil \Leftrightarrow x_0  +\lfloor\frac{n}{2}\rfloor> 2n-1-x_1
$$

###### Protocol

Over$\ Z$, $x_0, x_1\in \{2^l\}$

We can use 2's complement to encode $a_0,a_1$, where $a = a_0\oplus a_1$

![1730124171932](image/ctf2/1730124171932.png)

It is obvious that the result of DReLU can be deduced by the MSB of a. (the result is the inversion of the MSB for a)

    Step1:  we can divide a0 and a1 into 2 parts

    Step2: online, we only compare$2^{l-1} - 1 - x0$ and $x_1$ by Millionares'

    Step3: offline, we generate$[DReLU]_b = msb_b\oplus[carry]_b\oplus b$

The last b is used as an inversion of the MSB for a.

Over$\ Z_n$,

![1730124263632](image/ctf2/1730124263632.png)

In the first 3 steps, we obtain the share of lt, wrap, and rt

    Step 4: if wrap = 1, z = lt XOR rt; else, z = 0, we can interpret as, [0,0,1,0]

    Step 5: Pb = z XOR lt XOR b , we can interpret as, [1,0,1,0]

![1730120881231](https://file+.vscode-resource.vscode-cdn.net/h%3A/%E5%A4%87%E4%BB%BDG%E7%9B%98/EDA/ctf2/image/ctf2/1730120881231.png)

###### Optimization

1. First let $P_1$ adjust the input of step 1, 2, 3 to be consistent, so that the leaf nodes in $F_{Mill}$ can be calculated together (step 9 & 10 algorithm 1). The specific method is to add $\frac{n-1}{2}$, to $P_1$ 's input of step 1&3 in algorithm 3;
2. Further, the $P_0$ reduces the execution of step 2 or step 3 according to its input. That is, if $\lang a\rang_0^n>\frac{n-1}{2}$, then $it = 1$ in step 2; Otherwise, $rt = 0$ in step 3.

    Correctness can be easily deduced if we take the input of$P_1$ in step14 into the OT protocol from step 5 to step 12.

    Since$\lang z\rang_0^B$ is completely random, the safety can be proved by $(F_{Mill}^{\eta+1},\begin{pmatrix}4\\1\end{pmatrix}-OT)$-hybird model.

    Algorithm 4 calls$\ F_{Mill}^{\eta+1}$ twice and $\ \begin{pmatrix}4\\1\end{pmatrix}-OT$ once. Total communication < $\frac{3}{2}\lambda(\eta+1) + 28(\eta + 1) + 2\lambda +4$, which is better than algorithm 3$\times$.

![1730128177673](image/ctf2/1730128177673.png)

##### ReLU

Relu = DRelu * a. We can use DRelu's and Mux's protocol to caculate Relu

![1730280095231](image/ctf2/1730280095231.png)

##### Maxpool layer

A pairwise comparison is used to select the largest method. Choose the two largest numbers first, and then compare them in turn.

For x,y are both at P0,P1 as the number of secret shares, first locally calculate the respective secret share x-y=w. Input the two secret shared w into the DRelu protocol to see if w is greater than 0, and the result is set to v. Using a data selector, t=wv. Finally each output z=y+t.

Consider that when v=0, t=0, the output z=y.

Consider v=1, t=w=x-y The output z=y+x-y=x.

### Part 4 Division and Truncation (linear)

##### 4.1 Expressing general division and truncation using arithmetic over secret shares

    Define$\ idiv:\mathbb{Z}\times\mathbb{Z}\rightarrow\mathbb{Z}$ as signed integer division, the quotient is prone to $-\infty$, the remainder have the same sign as divisor. Furthermore, define:

$$
rdiv(a,d)=idiv(a_u-1\{a_u\geq\lceil n/2\rceil\}\cdot n,d)\ mod\ n,
$$

    Here$\ a_u\in\{0,...,n-1\}$ is the unsigned exhibition for $a\in\mathbb{Z}_n$,  $0<d<n$.

###### Theorem 4.1.1

    Let$\ a\in\mathbb{Z}_n$ has its share as $\lang a\rang^n_0,\lang a\rang^n_1\in \mathbb{Z}_n$, $n = n^1\cdot d+n^0$. ($n^0,n^1,d\in\mathbb{Z}_n$ and $0\leq a_0^0,a_1^0<d)$.  Denote $n' = \lceil \frac{n}{2}\rceil\in\mathbb{Z}$. Define $corr,\ A,\ B,\ C$ as follows:

$$
corr = \begin{cases}-1,&(a_u\geq n')\wedge(a_0< n')\wedge(a_1<n')\\1,&(a_u< n')\wedge(a_0\geq n')\wedge(a_1\geq n')\\0,&otherwise\end{cases}
$$

$$
A = a_0^0 + a_1^0-(1\{a_0\geq n'\} + 1\{a_1\geq n'\})\cdot n^0
$$

$$
B = idiv(a_0^0-1\{a_0\geq n'\}\cdot n',d) + idiv(a_1^0-1\{a_1\geq n'\}\cdot n',d)
$$

$$
C = 1\{A<d\} + 1\{A<0\} + 1\{A<-d\}.
$$

Therefore we have:

$$
rdiv(\lang a\rang_0^n,d) + rdiv(\lang a\rang_1^n,d) + (corr\cdot n ^1 + 1-C-B)=_n rdiv(a,d)
$$

Proof 

* decompose $rdiv(\lang a\rang_i^n,d)$;![1730807749942](image/ctf2/1730807749942.png)
* $a_u = a_0 + a_1-w\cdot n$, where $w = 1\{a_0 + a_1\geq n\}$;![1730807790545](image/ctf2/1730807790545.png)
* Since $\leq a_0^0 + a_1^0-w\cdot n^0-k\cdot d<d$��![1730807821489](image/ctf2/1730807821489.png)
* Then we have:![1730807839895](image/ctf2/1730807839895.png)
* Let $A_i' = idiv(a_0^0,a_1^0-i\cdot n^0,d)$;![1730807855207](image/ctf2/1730807855207.png)

    From this table we know that$\ c^1 = corr$. Therefore $c =_n corr\cdot n^1+c^0 - B.$

    Let$\ C_0 = 1\{A<d\}$, $\ C_1 = 1\{A<0\}$, $\ C_0 = 1\{A<-d\}$, then $C = C_0 + C_1 + C_2$

    Row 1 corresponds to$A = a_0^0 + a_1^0$;

    Row 8 corresponds to$A = a_0^0 + a_1^0 - 2\cdot n^0$;

    Other rows correspond to$c^0 = idiv(A,d)$;

    It is obvious that$-2\cdot d+2\leq A\leq2\cdot d-2$, therefore $c^0\in\{-2,-1,0,1\}$

    Therefore,$c = _n corr\cdot n^1 + (1-C)-B$

Corollary 4.2: truncation for $l-bit$ integers can be simplified to:

$$
(a_0>>s) + (a_1>>s) + corr\cdot 2^{l-s} + 1\{a_0^0 + a_1^0\geq2^s\} = _L(a>>s)
$$

##### 4.2 Truncation of $l-bit$ Integer

    $F_{Trunc}^{int,l,s}$ is a function that performs on$\ l-bit$ integers and truncate its lower s bits. The result is exactly the same as cleartext.

    In the algorithm, step 1-15 calculate$\ corr$. Step 16 calculate $1\{a_0^0 + a_1^0\geq2^s\}$, and conduct $F_{B2A}$ at step 17.

    The complicated part is the verification of$\ corr$, especially the construction of $OT$ in step 15. By taking $P_1$'s input $\lang m\rang_1^m||x_1$ into step 15, we can verify its correctness. Since the calculation of $\ corr $ is completely random, the saftey can be proved by $(F_{DReLU}^{int,l}, \begin{pmatrix}4\\1\end{pmatrix}-OT, F_{Mill}^{s}, F_{B2A}^L)$-hybird model.

    Since$\ (F_{DReLU}^{int,l}, \begin{pmatrix}4\\1\end{pmatrix}-OT, F_{Mill}^{s}, F_{B2A}^L)$ are each conducted once, the communicationis less than $\lambda l + 2\lambda + 19l + communication\ for\ F_{Mill}^s$, which is mainly based on s.

![1730805408424](image/ctf2/1730805408424.png)

##### 4.3 Division

    $F_{Div}^{ring,n,d}$  stands for division on general ring. This protocol is similar to truncation protocol.  Since $-3d+2\leq A-d,A,A+d\leq3d-2$,  $C=(DReLU(A-d)\oplus1) + (DReLU(A+d)\oplus1)$ can be calculated in terms of $\delta = \lceil log_2^{6d}\rceil-bit$. Before calculate C, A needs to be calculated first. Therefore, calculate A on $\mathbb{Z}_n$ and $\mathbb{Z}_\Delta(\Delta = 2^\delta)$ simultaneously.

![1730805908762](image/ctf2/1730805908762.png)

    Correctness verification is the same to truncation's. Safety check is based on$(\begin{pmatrix}4\\1\end{pmatrix}-OT_{\eta + \delta}, F_{DReLU}^{ring,n}, F_{B2A}^n)$-hybird model.

    The protocol calls$\begin{pmatrix}4\\1\end{pmatrix}-OT_{\eta + \delta}\ and\  F_{DReLU}^{ring,n}$ once, $F_{DReLU}^{\delta},\ and\  F_{B2A}^n$ 3 times. Total communication is less than $(\frac{3}{4}\lambda + 34)\cdot(\eta + 2\delta)$ bit.

    Improvement on Avgpool is listed below:

![1730806256949](image/ctf2/1730806256949.png)

##### 4.4 Truncation Optimization

    For scenarios where$\ 2\cdot n^0\leq d = 2^s$ is satisfied, $\ A\geq -d$ always stands. Therefore $\ A\geq -d$ in the calculation of Ccan be omitted. Further decrease by $\delta = \lceil log_2(4d)\rceil$.

### Part 5 Secure Inference

The inference process of neural network model is carried out as follows

1. Linear layer: Call multiplication based on OT or multiplication based on HE, adjust according to the scene;
2. ReLU: invokes the ReLU protocol
3. Avgpool: Call division protocol
4. Truncation: Since the output of ReLU is non-negative, we can reduce calculation expense
5. Maxpool and Argmax: Millionaire's and $F_{MUX}$ in order

### Part 6 Evaluation

    In this paper, they implement OT based on EMP and chooses efficient AES. The HE is SEAL. They integrate those into CrypTFlow system. Firstly, the ReLU is calculated bu comparing with the GC-based method, raising it by 2-25$\times$.

![1730799028461](image/ctf2/1730799028461.png)

    Next, the ReLU in the real network is promoted as follows. In the local area network, improve the calculation 8-9$\times$, in the wide area network to improve the time 18-21$\times$, improve the communication 7-9$\times$.

![1730799013804](image/ctf2/1730799013804.png)

    For Avgpool, time is improved 51$\times$.

![1730799203812](image/ctf2/1730799203812.png)

    Compared with Delphi, it is still greatly improved in both nonlinear layer and online calculation.

![1730799135447](image/ctf2/1730799135447.png)

    Finally, for large networks, predictions can be made in 10min (LAN) and 20min (WAN).

![1730799363085](image/ctf2/1730799363085.png)

### Part 7 Conclusion

    This paper is one of the best two-party computational comparison protocols and is the basis of SIRNN. In this paper, an efficient calculation method of accurate truncation and division is presented. This ideas are very enlightening for later work.

### Appendix I: Code-reading

###### millionaire_with_equality.h:

[`MillionaireWithEquality`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A28%2C%22character%22%3A29%7D%7D%5D%2C%229f65f91c-63ec-4475-aaf7-890aae28b581%22%5D "Go to definition") ��һ��ģ���࣬ʹ��ģ����� [`IO`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A28%2C%22character%22%3A19%7D%7D%5D%2C%229f65f91c-63ec-4475-aaf7-890aae28b581%22%5D "Go to definition")����������ҪĿ����ʵ�ְ���������ıȽ�Э�飬����֧������Լ�顣��Ĺ�����Ա�������£�

- [`IO *io`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A28%2C%22character%22%3A19%7D%7D%5D%2C%229f65f91c-63ec-4475-aaf7-890aae28b581%22%5D "Go to definition"): һ��ָ�� IO �����ָ�룬���ڴ����������������
- `sci::OTPack<IO> *otpack`: һ��ָ�� `sci::OTPack<IO>` �����ָ�룬���ڴ��� Oblivious Transfer��OT��Э�顣
- [`TripleGenerator<IO> *triple_gen`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A32%2C%22character%22%3A2%7D%7D%5D%2C%229f65f91c-63ec-4475-aaf7-890aae28b581%22%5D "Go to definition"): һ��ָ�� [`TripleGenerator<IO>`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A32%2C%22character%22%3A2%7D%7D%5D%2C%229f65f91c-63ec-4475-aaf7-890aae28b581%22%5D "Go to definition") �����ָ�룬����������Ԫ�飬��Щ��Ԫ����Э�������ڰ�ȫ���㡣
- [`MillionaireProtocol<IO> *mill`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A33%2C%22character%22%3A2%7D%7D%5D%2C%229f65f91c-63ec-4475-aaf7-890aae28b581%22%5D "Go to definition"): һ��ָ�� [`MillionaireProtocol<IO>`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A33%2C%22character%22%3A2%7D%7D%5D%2C%229f65f91c-63ec-4475-aaf7-890aae28b581%22%5D "Go to definition") �����ָ�룬����ִ�а����̱Ƚ�Э�顣
- [`bool del_mill`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A48%2C%22character%22%3A6%7D%7D%5D%2C%229f65f91c-63ec-4475-aaf7-890aae28b581%22%5D "Go to definition"): һ������ֵ��ָʾ�Ƿ���Ҫ������������ɾ�� [`mill`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A49%2C%22character%22%3A6%7D%7D%5D%2C%229f65f91c-63ec-4475-aaf7-890aae28b581%22%5D "Go to definition") ����
- [`int party`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A40%2C%22character%22%3A30%7D%7D%5D%2C%229f65f91c-63ec-4475-aaf7-890aae28b581%22%5D "Go to definition"): ��ʾ���뷽�ı�ţ�ͨ����������Э���еĲ�ͬ�����ߡ�
- [`int l, r, log_alpha, beta, beta_pow`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A62%2C%22character%22%3A36%7D%7D%5D%2C%229f65f91c-63ec-4475-aaf7-890aae28b581%22%5D "Go to definition"): ��Щ�����������ڴ洢Э���еĸ��ֲ������м�����������磬[`l`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A62%2C%22character%22%3A36%7D%7D%5D%2C%229f65f91c-63ec-4475-aaf7-890aae28b581%22%5D "Go to definition") ��ʾ���س��ȣ�[`beta`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A62%2C%22character%22%3A40%7D%7D%5D%2C%229f65f91c-63ec-4475-aaf7-890aae28b581%22%5D "Go to definition") ��ʾ������[`beta_pow`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A178%2C%22character%22%3A42%7D%7D%5D%2C%229f65f91c-63ec-4475-aaf7-890aae28b581%22%5D "Go to definition") ��ʾ [`beta`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A62%2C%22character%22%3A40%7D%7D%5D%2C%229f65f91c-63ec-4475-aaf7-890aae28b581%22%5D "Go to definition") ���ݡ�
- [`int num_digits, num_triples, log_num_digits`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A64%2C%22character%22%3A34%7D%7D%5D%2C%229f65f91c-63ec-4475-aaf7-890aae28b581%22%5D "Go to definition"): ��Щ�����������ڴ洢Э���еĸ��ּ���ֵ�����磬[`num_digits`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A64%2C%22character%22%3A34%7D%7D%5D%2C%229f65f91c-63ec-4475-aaf7-890aae28b581%22%5D "Go to definition") ��ʾ���ֵĸ�����[`num_triples`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A277%2C%22character%22%3A25%7D%7D%5D%2C%229f65f91c-63ec-4475-aaf7-890aae28b581%22%5D "Go to definition") ��ʾ��Ԫ���������
- [`uint8_t mask_beta, mask_r`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A169%2C%22character%22%3A51%7D%7D%5D%2C%229f65f91c-63ec-4475-aaf7-890aae28b581%22%5D "Go to definition"): ��Щ 8 λ�޷����������ڴ洢����ֵ��������Э���н���λ������

��Щ��Ա������ͬ������ [`MillionaireWithEquality`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A28%2C%22character%22%3A29%7D%7D%5D%2C%229f65f91c-63ec-4475-aaf7-890aae28b581%22%5D "Go to definition") ��Ļ���������ʵ�ְ���������ıȽ�Э�������Լ�顣

 [`MillionaireWithEquality`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A40%2C%22character%22%3A2%7D%7D%5D%2C%2218b03e61-2d76-407c-a38e-1e2da2e2ce10%22%5D "Go to definition") ��Ĺ��캯�������ڳ�ʼ�����ʵ�������������������

- [`party`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A40%2C%22character%22%3A30%7D%7D%5D%2C%2218b03e61-2d76-407c-a38e-1e2da2e2ce10%22%5D "Go to definition"): ��ʾ���뷽�ı�š�
- [`io`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A40%2C%22character%22%3A41%7D%7D%5D%2C%2218b03e61-2d76-407c-a38e-1e2da2e2ce10%22%5D "Go to definition"): һ��ָ�� IO �����ָ�룬���ڴ����������������
- [`otpack`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A40%2C%22character%22%3A62%7D%7D%5D%2C%2218b03e61-2d76-407c-a38e-1e2da2e2ce10%22%5D "Go to definition"): һ��ָ�� `sci::OTPack<IO>` �����ָ�룬���ڴ��� Oblivious Transfer��OT��Э�顣
- [`mill_in`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A41%2C%22character%22%3A51%7D%7D%5D%2C%2218b03e61-2d76-407c-a38e-1e2da2e2ce10%22%5D "Go to definition"): һ����ѡ��ָ�� [`MillionaireProtocol<IO>`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A41%2C%22character%22%3A26%7D%7D%5D%2C%2218b03e61-2d76-407c-a38e-1e2da2e2ce10%22%5D "Go to definition") �����ָ�룬������ṩ����Ĭ��Ϊ [`nullptr`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A41%2C%22character%22%3A61%7D%7D%5D%2C%2218b03e61-2d76-407c-a38e-1e2da2e2ce10%22%5D "Go to definition")��
- [`bitlength`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A42%2C%22character%22%3A30%7D%7D%5D%2C%2218b03e61-2d76-407c-a38e-1e2da2e2ce10%22%5D "Go to definition"): һ����������ʾ���س��ȣ�Ĭ��Ϊ 32��
- [`radix_base`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A42%2C%22character%22%3A50%7D%7D%5D%2C%2218b03e61-2d76-407c-a38e-1e2da2e2ce10%22%5D "Go to definition"): һ����������ʾ������Ĭ��Ϊ [`MILL_PARAM`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A42%2C%22character%22%3A63%7D%7D%5D%2C%2218b03e61-2d76-407c-a38e-1e2da2e2ce10%22%5D "Go to definition")��

�ڹ��캯���У����Ƚ�����Ĳ�����ֵ����ĳ�Ա��������� [`mill_in`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A41%2C%22character%22%3A51%7D%7D%5D%2C%2218b03e61-2d76-407c-a38e-1e2da2e2ce10%22%5D "Go to definition") Ϊ [`nullptr`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A41%2C%22character%22%3A61%7D%7D%5D%2C%2218b03e61-2d76-407c-a38e-1e2da2e2ce10%22%5D "Go to definition")���򴴽�һ���µ� [`MillionaireProtocol<IO>`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A41%2C%22character%22%3A26%7D%7D%5D%2C%2218b03e61-2d76-407c-a38e-1e2da2e2ce10%22%5D "Go to definition") ���󣬲��� [`del_mill`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A48%2C%22character%22%3A6%7D%7D%5D%2C%2218b03e61-2d76-407c-a38e-1e2da2e2ce10%22%5D "Go to definition") ����Ϊ [`true`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A48%2C%22character%22%3A17%7D%7D%5D%2C%2218b03e61-2d76-407c-a38e-1e2da2e2ce10%22%5D "Go to definition")���Ա�������������ɾ���ö��󡣷��򣬽� [`mill_in`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A41%2C%22character%22%3A51%7D%7D%5D%2C%2218b03e61-2d76-407c-a38e-1e2da2e2ce10%22%5D "Go to definition") ��ֵ�� [`mill`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A49%2C%22character%22%3A6%7D%7D%5D%2C%2218b03e61-2d76-407c-a38e-1e2da2e2ce10%22%5D "Go to definition") ��Ա������

���������� [`mill`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A49%2C%22character%22%3A6%7D%7D%5D%2C%2218b03e61-2d76-407c-a38e-1e2da2e2ce10%22%5D "Go to definition") ����� [`triple_gen`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A52%2C%22character%22%3A10%7D%7D%5D%2C%2218b03e61-2d76-407c-a38e-1e2da2e2ce10%22%5D "Go to definition") ��Ա������ֵ����� [`triple_gen`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A52%2C%22character%22%3A10%7D%7D%5D%2C%2218b03e61-2d76-407c-a38e-1e2da2e2ce10%22%5D "Go to definition") ��Ա��������󣬵��� [`configure`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A53%2C%22character%22%3A4%7D%7D%5D%2C%2218b03e61-2d76-407c-a38e-1e2da2e2ce10%22%5D "Go to definition") ������ʹ�ô���� [`bitlength`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A42%2C%22character%22%3A30%7D%7D%5D%2C%2218b03e61-2d76-407c-a38e-1e2da2e2ce10%22%5D "Go to definition") �� [`radix_base`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A42%2C%22character%22%3A50%7D%7D%5D%2C%2218b03e61-2d76-407c-a38e-1e2da2e2ce10%22%5D "Go to definition") �����������á�

������캯��ȷ���� [`MillionaireWithEquality`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A40%2C%22character%22%3A2%7D%7D%5D%2C%2218b03e61-2d76-407c-a38e-1e2da2e2ce10%22%5D "Go to definition") ���ʵ���ڴ���ʱ��ȷ��ʼ�����б�Ҫ�ĳ�Ա��������������Ҫ������ʹ�����е� [`MillionaireProtocol<IO>`](command:_github.copilot.openSymbolFromReferences?%5B%22%22%2C%5B%7B%22uri%22%3A%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fezpc_dir%2FEzPC%2FSCI%2Fsrc%2FMillionaire%2Fmillionaire_with_equality.h%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%2C%22pos%22%3A%7B%22line%22%3A41%2C%22character%22%3A26%7D%7D%5D%2C%2218b03e61-2d76-407c-a38e-1e2da2e2ce10%22%5D "Go to definition") ����

[`bitlen_lt_beta`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) ��һ�����ڱȽϵĺ������Ƚ��������뷽�����ݣ������رȽϽ�����ú���ʹ���˰�ȫ�෽���㣨MPC���е� Oblivious Transfer��OT��Э����ʵ�ְ���������ıȽϡ������Ĳ������£�

* [`res_cmp`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html): һ��ָ�� [`uint8_t`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) ���������ָ�룬���ڴ洢�ȽϽ����
* [`res_eq`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html): һ��ָ�� [`uint8_t`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) ���������ָ�룬���ڴ洢����Լ������
* [`data`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html): һ��ָ�� [`uint64_t`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) ���������ָ�룬����Ҫ�Ƚϵ����ݡ�
* [`num_cmps`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html): һ����������ʾҪ���бȽϵĴ�����
* [`bitlength`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html): һ����������ʾÿ�����ݵı��س��ȡ�
* [`greater_than`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html): һ������ֵ��ָʾ�Ƚϵķ���Ĭ��Ϊ [`true`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html)��
* [`radix_base`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html): һ����������ʾ������Ĭ��Ϊ [`MILL_PARAM`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html)��

�������ȼ��� [`N`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) �� [`mask`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html)������ [`N`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) �� 2 �� [`bitlength`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) ���ݣ�[`mask`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) �� [`N - 1`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html)����Щֵ���������������ȷ��������ָ���ı��س����ڡ�

���������������ݲ��뷽�Ľ�ɫ��[`party`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html)�����в�ͬ�Ĳ�������� [`party`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) �� `sci::ALICE`������������������ [`res_cmp`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) �� [`res_eq`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) ���顣Ȼ��Ϊÿ���Ƚϴ���һ�� [`leaf_messages`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) ���飬������ [`set_leaf_ot_messages`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) ��������Ҷ�ڵ�� OT ��Ϣ������ [`bitlength`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) ��ֵ��ѡ��ͬ�� OT Э�鷢����Щ��Ϣ������ͷ� [`leaf_messages`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) ������ڴ档

��� [`party`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) �� `sci::BOB`���򴴽�һ�� [`choice`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) ���飬�洢ÿ�����ݵ�����ֵ������ [`bitlength`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) ��ֵ��ѡ��ͬ�� OT Э����ձȽϽ����Ȼ�󣬸��� [`res_eq`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) �� [`res_cmp`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) ���飬�ͷ� [`choice`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) ������ڴ档

�������ͨ��ʹ�� OT Э�������������ɣ�ȷ���˱ȽϹ��̵İ�ȫ�Ժ���˽�ԡ�

[`compare_with_eq`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) �������ڱȽ��������뷽�����ݣ������رȽϽ��������Լ�������ú���ʹ�ð�ȫ�෽���㣨MPC���е� Oblivious Transfer��OT��Э����ʵ�ְ���������ıȽϡ������Ĳ������£�

* [`res_cmp`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html): һ��ָ�� [`uint8_t`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) ���������ָ�룬���ڴ洢�ȽϽ����
* [`res_eq`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html): һ��ָ�� [`uint8_t`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) ���������ָ�룬���ڴ洢����Լ������
* [`data`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html): һ��ָ�� [`uint64_t`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) ���������ָ�룬����Ҫ�Ƚϵ����ݡ�
* [`num_cmps`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html): һ����������ʾҪ���бȽϵĴ�����
* [`bitlength`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html): һ����������ʾÿ�����ݵı��س��ȡ�
* [`greater_than`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html): һ������ֵ��ָʾ�Ƚϵķ���Ĭ��Ϊ [`true`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html)��
* [`radix_base`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html): һ����������ʾ������Ĭ��Ϊ [`MILL_PARAM`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html)��

�������ȵ��� [`configure`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) ���������ݴ���� [`bitlength`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) �� [`radix_base`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) �����������á���� [`bitlength`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) С�ڻ���� [`beta`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html)������� [`bitlen_lt_beta`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) �������бȽϣ������ؽ����

������������ȷ�� [`num_cmps`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) �� 8 �ı�����������ǣ��������䡣Ȼ�󣬺���Ϊÿ���Ƚϴ���һ����չ�������� [`data_ext`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html)������ԭʼ���ݸ��Ƶ��������У���䲿���� 0 ���롣

�������ŷ����ڴ棬���ڴ洢����������ȡ�Ļ������֣�[`digits`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html)�����ȽϽ����[`leaf_res_cmp`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html)��������Լ������[`leaf_res_eq`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html)����Ȼ�󣬺�������������ȡ�������֣����洢�� [`digits`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) �����С�

���ݲ��뷽�Ľ�ɫ��[`party`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html)�����������в�ͬ�Ĳ�������� [`party`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) �� `sci::ALICE`������������������ [`leaf_res_cmp`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) �� [`leaf_res_eq`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) ���飬������ [`set_leaf_ot_messages`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) ��������Ҷ�ڵ�� OT ��Ϣ��Ȼ�󣬸��� [`bitlength`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) �� [`r`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) ��ֵ��ѡ��ͬ�� OT Э�鷢����Щ��Ϣ������ͷ� [`leaf_ot_messages`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) ������ڴ档

��� [`party`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) �� `sci::BOB`������� [`bitlength`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) �� [`r`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) ��ֵ��ѡ��ͬ�� OT Э����ձȽϽ����Ȼ�󣬸��� [`leaf_res_eq`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) �� [`leaf_res_cmp`](vscode-file://vscode-app/d:/Program%20Files%20(x86)/Microsoft%20VS%20Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.esm.html) ���飬�ͷ� `choice` ������ڴ档

�������ͨ��ʹ�� OT Э�������������ɣ�ȷ���˱ȽϹ��̵İ�ȫ�Ժ���˽�ԡ�
