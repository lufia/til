==========
Plan 9
==========

.. highlight:: c

呼び出し規約
=============

* http://herumi.in.coocan.jp/prog/x64.html
* https://9p.io/sys/doc/asm.html
* https://qiita.com/edo_m18/items/83c63cd69f119d0b9831

スタック
--------

スタックに積まれる1つの値が持つサイズは、アーキテクチャに依存する。
x86の場合は4byteだが、x64の場合は8byteで固定。

スタックに積むと、アドレスの高い方から低い方へ増えて行く。

呼び出された側は ``x(FP)`` で取り出せばいいが、
呼び出す側はどうやってスタックに積めばいいんだろう。

このコードをアセンブリした::

	long long
	calc(int n, char c)
	{
		long long v1, v2, *v;
	
		v = &v1;
		*v = 100;
		v = &v2;
		*v = 200;
		return n+c+v1+v2;
	}

	void
	main(void)
	{
		long long r, *v;
	
		r = calc(20, 1);
		v = &r;
		*v = 1;
	}

.. code-block:: asm

``6c -S`` の結果::

	TEXT calc+0(SB),0,$24
		// 8(FP) = c
		// 0(FP) = n
		// -8(SP) = v1
		// -16(SP)= v2
		// -24(SP)= v?
		LEAQ	v1+-8(SP),AX
		MOVQ	$100,(AX)
		LEAQ	v2+-16(SP),AX
		MOVQ	$200,(AX)
		MOVBLSX	c+8(FP),AX
		ADDL	BP,AX
		MOVLQSX	AX,AX
		ADDQ	v2+-16(SP),AX
		ADDQ	v1+-8(SP),AX
		RET

	TEXT main+0(SB),0,$32
		// 24(SP)= r
		// 16(SP)= v
		// 8(SP) = arg1:n
		// 0(SP) = arg2:c -> BP
		MOVL	$20,BP
		MOVL	$1,CX
		MOVL	CX,8(SP)
		CALL	calc+0(SB)
		MOVQ	AX,r+-8(SP)
		LEAQ	r+-8(SP),AX
		MOVQ	$1,(AX)
		RET

レジスタ
========

AX, BX, CX, DX, DI, SI(8a, 6a)
	386のレジスタ

R8..R15(6a)
	x64で増えたレジスタ

PC(8a, 6a)
	プログラムカウンタ

	EIP, RIPに相当

BP(8a, 6a)
	スタックベースポインタレジスタ

SB(8a, 6a)
	スタティック領域の先頭を指す擬似レジスタ

SP(8a, 6a)
	スタックの先頭を指す擬似レジスタ

	ESP, RSPに相当

FP(8a, 6a)
	フレームの先頭を指す擬似レジスタ

F0..F7(8a)
	387の浮動小数点レジスタ

M0..M7(6a)
	MMXレジスタ

X0..X15(6a)
	XMMレジスタ

Y0..Y15(6a)
	YMMレジスタ

* `x86/x86_64関数呼び出しチートシートを書いた <http://d.sunnyone.org/2012/09/x86x8664.html>`_
* `x86の浮動小数計算とSIMD命令の変遷 <https://qiita.com/lpha_z/items/eafa9c13532c9ac80d4b>`_

``MOVx`` 命令などの **x** によってレジスタのサイズが変わるので、
``EAX`` や ``RAX`` などは意識しなくて良いが、
基本的に下位ビットが使われるので ``AH`` などを使いたい場合は明記する必要がある。

= ======= === ========
x 名前    bit レジスタ   
= ======= === ========
B byte    8   AL
W word    16  AX
L long    32  EAX
Q quad    64  RAX
O octword 128 ?
= ======= === ========
