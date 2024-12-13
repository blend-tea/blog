---
title: "Windows APIをばれないように呼び出したい" 
slug: "want-to-call-windows-api-stealthily"
---
ゲームとかアプリを作るときって、やっぱ解析されづらい最強バイナリ作りたいよね～

デバッグ検知とかだとIsDebuggerPresentだったりNtSetInformationThreadだったり使ってデバッガ対策するのが淑女の嗜みだけど
普通にWindows APIを呼び出そうものなら簡単にばれてしまう
# 普通にWindows APIを呼び出す
```rust
#[cfg(windows)]
extern crate winapi;


#[cfg(windows)]
fn main() {
    use winapi::um::debugapi::IsDebuggerPresent;
    unsafe {
        if IsDebuggerPresent() != 0 {
            println!("Debugger detected");
        } else {
            println!("No debugger detected");
        }
    }
}
```
これを適当にコンパイルしてGhidraで解析すると
```c
void UndefinedFunction_14000100c(undefined8 param_1,undefined8 param_2)

{
  BOOL BVar1;
  undefined **ppuStack_30;
  undefined8 uStack_28;
  undefined8 uStack_20;
  undefined auStack_18 [16];
  
  BVar1 = IsDebuggerPresent();
  if (BVar1 == 0) {
    ppuStack_30 = &PTR_s_No_debugger_detected_1400173f8;
  }
  else {
    ppuStack_30 = &PTR_s_Debugger_detected_140017420;
  }
  uStack_28 = 1;
  uStack_20 = 8;
  auStack_18 = ZEXT816(0);
  FUN_140010e20((longlong *)&ppuStack_30,param_2);
  return;
}

```
とIsDebuggerPresentを使っているのがばれる。

# LoadLibraryを使う
LoadLibraryとGetProcAddressを使う
```rust
#[cfg(windows)]
use winapi::um::libloaderapi::{LoadLibraryA, GetProcAddress};
use std::ffi::CString;

macro_rules! cstr {
    ($str:expr) => {
        CString::new($str).unwrap()
    };
}
#[cfg(windows)]
fn main() {
    unsafe {
        let h_module = {
            let file_name = cstr!("kernel32.dll");
            LoadLibraryA(file_name.as_ptr())
        };
        let p_is_debugger_present = {
            let func_name = cstr!("IsDebuggerPresent");
            GetProcAddress(h_module, func_name.as_ptr())
        };
        let is_debugger_present_imported = std::mem::transmute::<_, fn() -> i32>(p_is_debugger_present);
        if is_debugger_present_imported() != 0 {
            println!("Debugger is present");
        } else {
            println!("Debugger is not present");
        }
    }
}
```
Ghidra
```c

void UndefinedFunction_1400011db(void)

{
  code *pcVar1;
  LPCSTR lpLibFileName;
  HMODULE hModule;
  FARPROC pFVar2;
  INT_PTR IVar3;
  longlong lVar4;
  undefined4 uStack_78;
  undefined4 uStack_74;
  undefined4 uStack_70;
  undefined4 uStack_6c;
  undefined4 uStack_68;
  undefined4 uStack_64;
  undefined **ppuStack_58;
  undefined8 uStack_50;
  undefined8 uStack_48;
  undefined auStack_40 [16];
  
  FUN_1400016e0((longlong *)&uStack_78,"kernel32.dll<redacted>",(void *)0xc);
  if (CONCAT44(uStack_74,uStack_78) == -0x8000000000000000) {
    lpLibFileName = (LPCSTR)CONCAT44(uStack_6c,uStack_70);
    lVar4 = CONCAT44(uStack_64,uStack_68);
    hModule = LoadLibraryA(lpLibFileName);
    FUN_1400010d7(lpLibFileName,lVar4);
    FUN_1400016e0((longlong *)&uStack_78,"IsDebuggerPresentDebugger is not present\n",(void *)0x11);
    if (CONCAT44(uStack_74,uStack_78) == -0x8000000000000000) {
      lVar4 = CONCAT44(uStack_64,uStack_68);
      pFVar2 = GetProcAddress(hModule,(LPCSTR)CONCAT44(uStack_6c,uStack_70));
      FUN_1400010d7((LPCSTR)CONCAT44(uStack_6c,uStack_70),lVar4);
      IVar3 = (*pFVar2)();
      if ((int)IVar3 == 0) {
        ppuStack_58 = (undefined **)&DAT_140017468;
      }
      else {
        ppuStack_58 = &PTR_s_Debugger_is_present_140017490;
      }
      uStack_50 = 1;
      uStack_48 = 8;
      auStack_40 = ZEXT816(0);
      FUN_1400112d0((longlong *)&ppuStack_58,lVar4);
      return;
    }
  }
  FUN_1400159f0("called `Result::unwrap()` on an `Err` value",0x2b,&ppuStack_58,&PTR_LAB_1400173e0,
                &DAT_140017420);
  pcVar1 = (code *)swi(3);
  (*pcVar1)();
  return;
}
```
これだけでもかなり分かりづらくなった。でもIsDebuggerPresentの文字があるのが気になる。
# 文字列難読化する
obfstr crateを使う
```rust
#[cfg(windows)]
use winapi::um::libloaderapi::{LoadLibraryA, GetProcAddress};
use std::ffi::CString;
use obfstr::obfstr;

macro_rules! cstr {
    ($str:expr) => {
        CString::new($str).unwrap()
    };
}
#[cfg(windows)]
fn main() {
    unsafe {
        let h_module = {
            let file_name = cstr!(obfstr!("kernel32.dll"));
            LoadLibraryA(file_name.as_ptr())
        };
        let p_is_debugger_present = {
            let func_name = cstr!(obfstr!("IsDebuggerPresent"));
            GetProcAddress(h_module, func_name.as_ptr())
        };
        let is_debugger_present_imported = std::mem::transmute::<_, fn() -> i32>(p_is_debugger_present);
        if is_debugger_present_imported() != 0 {
            println!("Debugger is present");
        } else {
            println!("Debugger is not present");
        }
    }
}
```
Ghidra
長いので一部省略
```c

void UndefinedFunction_140001219(void)
{
  while (bVar8) {
    *(ulonglong *)(auStack_68 + lVar6) =
         ((ulonglong)(byte)(&DAT_140017430)[lVar6] | 0xb8c06533fd74dc00) ^
         *(ulonglong *)(lVar3 + lVar6);
    lVar6 = 8;
    bVar8 = false;
  }
  for (uVar7 = 8; uVar7 < 0xc; uVar7 = uVar7 + 4) {
    *(uint *)(auStack_68 + uVar7) =
         (uint)(byte)(&UNK_140017432)[uVar7] * 0x10000 + (uint)*(ushort *)(&DAT_140017430 + uVar7) +
         0x38000000 ^ *(uint *)(lVar3 + uVar7);
  }
  uStack_30 = (undefined4)uStack_60;
  lStack_38 = (longlong)auStack_68;
  FUN_140001820((longlong *)&uStack_88,&lStack_38,(void *)0xc);
  if (CONCAT44(uStack_84,uStack_88) == -0x8000000000000000) {
    lpLibFileName = (LPCSTR)CONCAT44(uStack_7c,uStack_80);
    lVar3 = CONCAT44(uStack_74,uStack_78);

    hModule = LoadLibraryA(lpLibFileName);

    FUN_1400010d7(lpLibFileName,lVar3);
    auStack_68 = (undefined  [8])0x140011192;
    auStack_68._0_4_ = 0xef43362b;
    lVar3 = FUN_1400011f5(0x140011192,0xef43362b);
    _auStack_68 = ZEXT816(0);
    for (uVar7 = 0; uVar7 < 0x10; uVar7 = uVar7 + 8) {
      *(ulonglong *)(auStack_68 + uVar7) =
           *(ulonglong *)((longlong)&DAT_14001743c + uVar7) ^ *(ulonglong *)(lVar3 + uVar7);
    }
    bStack_28 = *(byte *)(lVar3 + 0x10) ^ 0x65;
    lStack_38 = SUB168(_auStack_68,0);
    uStack_30 = (undefined4)uStack_60;
    uStack_2c = uStack_60._4_4_;
    FUN_140001820((longlong *)&uStack_88,&lStack_38,(void *)0x11);
    if (CONCAT44(uStack_84,uStack_88) == -0x8000000000000000) {

      pFVar4 = GetProcAddress(hModule,(LPCSTR)CONCAT44(uStack_7c,uStack_80));

    }
  return;
}
```
これで完璧？いいえまだです。

![debugger_result1](/blog/assets/images/2024-12-11-194016.png){: .align-center}

デバッガを使うとGetProcAddressの引数がばれてしまいます。

# GetProcAddressを改造
ループ処理でdllの関数名からハッシュ値を生成し比較。マッチしたら返す。ハッシュアルゴリズムにはfnv-1aの64bitを使用。ついでにマクロを作成。ちょっとした嫌がらせ程度にハッシュ関数をインライン展開。

**ちょっとした裏話：**最初はwindows-rs crateを使っていたんだけどIMAGE_NT_HEADERSとかがうまくうごかなかったり調子が悪そうだったのでwinapiに変更した。
{: .notice}

以下コード。長くなってきたので主な追加箇所のみ
```rust
macro_rules! func_to_hash {
    ($function_name:expr) => {
        fnv1a_64($function_name)
    };
}

fn get_proc_address(dll_handle: HMODULE, func_hash: u64) -> *mut __some_function {
    unsafe {
        let p_dos_hdr = dll_handle as *mut IMAGE_DOS_HEADER;
        let p_nt_hdr = (dll_handle as *mut u8).add((*p_dos_hdr).e_lfanew as usize) as *mut IMAGE_NT_HEADERS;
        let p_export_table = (dll_handle as *mut u8).add((*p_nt_hdr).OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_EXPORT as usize].VirtualAddress as usize) as *mut IMAGE_EXPORT_DIRECTORY;
        let number_of_names = (*p_export_table).NumberOfNames;
        let functions_address = (dll_handle as *mut u8).add((*p_export_table).AddressOfFunctions as usize) as *mut u32;
        let functions_names = (dll_handle as *mut u8).add((*p_export_table).AddressOfNames as usize) as *mut u32;
        let functions_ordinals = (dll_handle as *mut u8).add((*p_export_table).AddressOfNameOrdinals as usize) as *mut u16;

        let mut ret = null_mut();
        for i in 0..number_of_names {
            let name = (dll_handle as *mut u8).add(*functions_names.add(i as usize) as usize);
            let name = std::ffi::CStr::from_ptr(name as *const i8).to_string_lossy();
            if fnv1a_64(&name) == func_hash {
                ret = (dll_handle as *mut u8).add(*functions_address.add(*functions_ordinals.add(i as usize) as usize) as usize) as *mut __some_function;
            }
        }
        ret
    }
}

#[inline(always)]
fn fnv1a_64(s: &str) -> u64 {
    let mut hash: u64 = 0xcbf29ce484222325;
    for c in s.as_bytes() {
        hash ^= *c as u64;
        hash = hash.wrapping_mul(0x100000001b3);
    }
    hash
}
```
マッチしたタイミングでループを抜けるとループの直後にブレークポイントを設置するとばれた。

なのでループ脱出後にreturnするようにした。

![debugger_result2](/blog/assets/images/2024-12-11-201217.png){: .align-center}



# まとめ
これでCTFのrev問でもつくったら楽しそうだけど解けるのかな？