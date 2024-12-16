---
title: "TSGCTF2024-Writeup(rev 1問)"
slug: "tsgctf2024-writeup"
date: "2024-12-16"
---

# TSGCTF2024 Writeup
revを1問だけ解いた。
# Misbehave
## 問題文

{% capture notice-2 %}
**このバイナリ、少し変なんです……**

**初心者向けのヒント :**
- 添付ファイルはx86-64のLinux上で動くELF形式の実行可能ファイルです
- 実行して正しいFLAGを入力すると、Correct!と表示されます
- GhidraやIDA Freeで処理の概要を把握しましょう
- gdbで実際に動かしながら挙動を確認しましょう
- すべての処理の内容を正確に理解する必要はありません
    - その処理が何を入力とし、何を変更するのかを把握するだけで十分なこともあります
{% endcapture %}

<div class="notice">{{ notice-2 | markdownify }}</div>

## 解答
ヒント通りにいろんなツールでデコンパイルしてみる。IDA Freeが一番見やすいな～と思った

![IDA Decompiler](/blog/assets/images/2024-12-14-192113.png){: .align-center}

gen_randで生成した値とxorを取って4文字づつ比較している。ここからはデバッガの出番

gen_randが返す値を取得してみたら、最初らへんの値は同じだけどそれ以降は入力によって変化している気がした。

前の4文字が正しければ正しいgen_randを取得でき、次の4文字をgen_randとflag_encでxorをとることで復号できる。
なので正しい4文字を入力。gen_randを確認。8文字入力。gen_randを確認。12文字入力...という感じで正しいgen_randを取得しつつ復号する。
```bash
TSGC
TSGCTF{h
TSGCTF{h1dd3
TSGCTF{h1dd3n_fu
TSGCTF{h1dd3n_func7i
TSGCTF{h1dd3n_func7i0n_4
TSGCTF{h1dd3n_func7i0n_4nd_s
TSGCTF{h1dd3n_func7i0n_4nd_s31f_
TSGCTF{h1dd3n_func7i0n_4nd_s31f_g07_
TSGCTF{h1dd3n_func7i0n_4nd_s31f_g07_0ver
TSGCTF{h1dd3n_func7i0n_4nd_s31f_g07_0verwr17
TSGCTF{h1dd3n_func7i0n_4nd_s31f_g07_0verwr173}
```
もっとcleverなやり方があると思う。