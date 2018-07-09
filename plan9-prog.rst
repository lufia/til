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

	#include <u.h>
	#include <libc.h>

	vlong
	calc(int n, char c)
	{
		vlong v1, v2;

		v1 = 100;
		if(n > 0)
			calc(n-1, c);
		v2 = 200;
		print("n=%p, c=%p, v1=%p, v2=%p, v = %p\n", &n, &c, &v1, &v2);
		return v1+v2;
	}

	void
	main(void)
	{
		vlong r, n, *v;

		v = &n;
		*v = 2;
		r = calc(n, 1);
		*v += r;
	}

.. code-block:: asm

``6c -S`` の結果::

	TEXT calc+0(SB),0,$56
		//   8(FP) = c (第2引数) [d0]
		//   0(FP) = n (第1引数) [c8]
		//  32(SP) = print/v2 (第5引数)
		//  24(SP) = print/v1 (第4引数)
		//  16(SP) = print/c (第3引数)
		//   8(SP) = calc/c, print/n (第2引数)
		//   0(SP) = calc/n, print/fmt (第1引数だけど未使用)
		//  -8(SP) = v1 [b8]
		// -16(SP) = v2 [b0]
		MOVL	BP,n+0(FP)
		MOVQ	$100,v1+-8(SP)
		CMPL	n+0(FP),$0
		JLE		6(PC)
		MOVL	n+0(FP),BP
		DECL	BP
		MOVBLSX	c+8(FP),AX
		MOVL	AX,8(SP)
		CALL	calc+0(SB)
		MOVQ	$200,v2+-16(SP)
		DATA	.string<>+0(SB)/8,$"n=%p, c="
		DATA	.string<>+8(SB)/8,$"%p, v1=%"
		DATA	.string<>+16(SB)/8,$"p, v2=%p"
		DATA	.string<>+24(SB)/8,$", v = %p"
		MOVQ	$.string<>+0(SB),BP
		LEAQ	n+0(FP),AX
		MOVQ	AX,8(SP)
		LEAQ	c+8(FP),AX
		MOVQ	AX,16(SP)
		LEAQ	v1+-8(SP),AX
		MOVQ	AX,24(SP)
		LEAQ	v2+-16(SP),AX
		MOVQ	AX,32(SP)
		CALL	print+0(SB)
		MOVQ	v1+-8(SP),AX
		ADDQ	v2+-16(SP),AX
		RET

	TEXT main+0(SB),0,$40
		//   8(SP) = calc/c (第2引数)
		//   0(SP) = calc/n (第1引数だけどBPで渡すため未使用)
		// -16(SP) = n
		// -24(SP) = v
		LEAQ	n+-16(SP),AX
		MOVQ	AX,v+-24(SP)
		MOVQ	$2,(AX)
		MOVQ	n+-16(SP),BP
		MOVQL	BP,BP
		MOVL	$1,CX
		MOVL	CX,8(SP)
		CALL	calc+0(SB)
		MOVQ	AX,DX
		MOVQ	v+-24(SP),AX
		ADDQ	DX,(AX)
		RET
		DATA	.string<>+32(SB)/8,$"\n\z\z\z\z\z\z\z"
		GLOBL	.string<>+0(SB),$40
		END

.. code-block:: text

実行結果::

	n=7fffffffee48, c=7fffffffee50, v1=7fffffffee38, v2=7fffffffee30, v = c8
	n=7fffffffee88, c=7fffffffee90, v1=7fffffffee78, v2=7fffffffee70, v = c8
	n=7fffffffeec8, c=7fffffffeed0, v1=7fffffffeeb8, v2=7fffffffeeb0, v = c8

*n* と *v1* のアドレスは連続している？

``TEXT`` 命令の3番目で自動的に確保するスタックサイズを指定することができる。
関数呼び出し時に、指定した分のスタックが確保されているんだろう。
手で書く場合は0にすることが多いのは、必要な時にSPレジスタを操作できるから。

* `スタック領域の構成 <http://hack.ninja-web.net/academy003-060.htm>`_
* `コールスタックの仕組みを復習する <http://komaken.me/blog/2013/08/31/c言語コールスタックスタックフレームの仕組み/>`_

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
