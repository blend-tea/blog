---
title: "Binary Ninjaの使い方"
slug: "how-to-use-binaryninja"
date: "2024-12-25"
---

個人的にCTFでBinary Ninjaを使う機会が多いので、簡単な使い方をまとめておく。

# Binary Ninjaとは

Binary Ninjaはリバースエンジニアリングツールです。他にGhidraやIDA Proなどがあります。
Binary Ninjaは有償ソフトウェアですが、機能制限があるものの無料で使うことも可能です。
あと学割で安く買えます。

# インストール
無償版は以下のリンクから入手できます。

<a href="https://binary.ninja/free/">https://binary.ninja/free/</a>

# 使い方(静的解析編)
Binary Ninjaを起動すると以下のような画面になる。
![binaryninja_main](/blog/assets/images/2024-12-26-005000.png){: .align-center}

ここに解析したいファイルをドラッグアンドドロップするかFile -> Openを選択してファイルを開く。

![binaryninja_open](/blog/assets/images/2024-12-26-005400.png){: .align-center}
左のSymbolsはGhidraでいうところのSymbol Tree、中心がLinearでGhidraでいうところのDecompilerになる。
Linearウィンドウの左上のPE▼Linear▼High Level IL▼をいじるとGraphやTriage Summaryなどが見れる。ここら辺は実際にいじってみたほうがわかりやすい。
また左端と右端にあるアイコンを押すとSymbolやCross Referencesを表示/非表示にできる。
Shiftを押しながらクリックすると複数まとめて表示できる。

## よく使うショートカット

| ショートカット | 動作 |
|--------------|------|
| g | 指定したアドレスにジャンプ |
| n | 関数名やラベルの変更 |
| ; | コメントの追加 |
| Space | Linear/Graph表示の切り替え |
| i | Disassembly/High Level IL表示の切り替え |
| F5 | 疑似Cにデコンパイル |
| Tab | Disassemblyの表示 |
| Esc | 前の場所に戻る |
| Ctrl+z | 元に戻す |
| Ctrl+y | やり直し |

# 使い方(デバッグ編)
ツールバーのDebuggerからLaunchを選択 or 左端のDebuggerアイコンをクリックしデバッグを開始する or F6でデバッグを開始できる。

![binaryninja_debugger](/blog/assets/images/2024-12-26-011500.png){: .align-center}
## よく使うショートカット

| ショートカット | 動作 |
|--------------|------|
| F2 | ブレークポイント |
| F6 | デバッグ開始 |
| F7 | ステップオーバー(callがあった場合呼び出されている関数に移動) |
| F8 | ステップイン(callを1つの命令として実行) |
| F9 | 実行(ブレークポイントがなければ最後まで実行) |
| Ctrl+F9 | 現在の関数を抜けるまで実行 |
| g → rip | 現在の実行位置へ移動 |

## リモートデバッグ
ツールバーのDebuggerからDebug Adapter Settingsを選択。
Working DirectoryをリモートデバッグするOSのパスに書き換える。

![binaryninja_debug_adapter_settings](/blog/assets/images/2024-12-26-012400.png){: .align-center}

<a href="https://github.com/Vector35/debugger">https://github.com/Vector35/debugger</a>からBinary Ninja用のDebug Serverをダウンロードする。
plugins/lldb/bin/lldb-serverを起動する。
```bash
lldb-server p --server --listen 0.0.0.0:31337
```
起動したらBinary Ninja側でツールバーのDebuggerからConnect to Debug Serverを選択。
PlatformをLinuxの場合はremote-linuxを選択。あとはHostのIPとPortを入力しAcceptを押す。

リモートデバッグはIOで結構バグりがちな気がするのでローカルデバッグが確実
{: .notice--warning}

## 便利なところ
- Debuggerウィンドウ(左端の虫アイコン)/Debugger Info(右下のアイコン)が見やすい
- 右クリックメニューにあるパッチが便利
    - if文の条件を変えられる
- Graphが見やすい & 高級言語でもGraphが見れる(GhidraではAssemblyでしか見れない)
