# K12-AI-1st-Assignment-FirstOrderLogic

sse02-19 日付：19/06/16

## 課題１

以下の文 (a) ～ (e) を一階述語論理式に訳してください。変数記号以外（定数、関数、述語記号）は、指定した記号のみを用いてください。

利用可能な記号：

$$
veg(X)\,・・・「Xは野菜である」\\
eat(X)\,・・・「私はXを食べる」\\
p(X,Y)\,・・・「XはYの親である」
$$


#### (a) 「野菜以外（の食べもの）は何でも食べます」

- For all X, I eat X , if X are not vegetables. ：全てのXについて、Xが野菜でないなら、私はXを食べる。

$$
\forall X (\lnot veg(X) \rightarrow eat(X) )
$$

#### (b) 「食べる野菜も食べない野菜もあります」

- There exists X , which I do eat and X are vegetables.　：あるXが存在する、Xが野菜で、かつ、私はXを食べる。

- There exists Y , which I don’t eat  and Y are vegetables.　：あるYが存在する、Yが野菜で、かつ、私はYを食べない。
    $$
    \exists X \exists Y ((eat(X) \land veg(X)) \lor (\lnot eat(Y) \land veg(Y)))
    $$
    

#### (c)「どんな人にも、親がちょうど二人いる」

- There exists someone A,B, which is a parent  of X and A != B.
- There exists someone B,A, which is a parent  of X and A != B.
- For all C, A = C or B =C, if C is a parent 

$$
\exists A  \exists B \forall X ((p(A,X) \land \lnot (A=B)) \land (p(B,X) \and \lnot (A=B)) \land \forall C(p(C,X) \to ((A=C) \lor (B=C)))
$$

#### (d) 「k は m と nの公約数である」

- There exists some number A, which is (A*k = m).
- There exists some number B, which is (B*k = n).
- There exists some number A, B, which A is not equal to  B.

$$
\exists A  \exists B (((m = k*A) \land \lnot (A=B)) \and ((n=k*B) \land \lnot (B=A)))
\\ \equiv \exists A  \exists B ((m = k*A) \land (n=k*B) \land \lnot (A=B)) \equiv P(A,B,K)
$$

- 定数m,nに対して、For all k, k is 公約数, if 上記が成立する場合
    $$
    \forall K \forall A \forall B (P(A,B,K) \to 公約数(K,m,n))
    $$
    

#### (e) 「kは素数である」

- For all A, B,  A=1 and B=k , if A is 自然数 and B is 自然数 and  k/A =B

$$
\forall A\forall B ((自然数(A) \and 自然数(B) \and k/A =B) \to ((A=1) \and (B=K)) \equiv Q(A,B,K)
$$

- For all K, K is 素数 if 上記が成立する場合

$$
\forall K \forall A \forall B (Q(A,B,K)) \to 素数(K))
$$



以上