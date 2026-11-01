---
title: Windows PEB和TEB结构体概述
date: 2026-10-31 17:57:38
tags:
  - windows
categories: reverse
cover:
description:
permalink: posts/x86-peb-teb-overview/
---

## 摘要

PEB(进程环境块)与TEB(线程环境块)是windows中的重要数据结构，也是windows逆向工程和反调试相关的基础。下文主要研究环境为x86

## 正文

### TEB（Thread Environment Block，线程环境块）

TEB包含进程中运行线程的各种信息，比如栈边界，SEH链，TLS指针，所属进程的PEB指针等，进程中的每个线程都对应一个TEB结构体。
以下是MSDN官方关于TEB结构体的说明：
```
typedef struct _TEB {
  PVOID Reserved1[12];
  PPEB  ProcessEnvironmentBlock;
  PVOID Reserved2[399];
  BYTE  Reserved3[1952];
  PVOID TlsSlots[64];
  BYTE  Reserved4[8];
  PVOID Reserved5[26];
  PVOID ReservedForOle;
  PVOID Reserved6[4];
  PVOID TlsExpansionSlots;
} TEB, *PTEB;
```

其中 Reservedx 相关字段被遮盖，没有太多有用的信息。
本机的 x86 \_TEB结构大概如下：
```
typedef struct _TEB32 {
    NT_TIB32      NtTib;                           /* 0x000 */
    uint32_t      EnvironmentPointer;              /* 0x01C */
    CLIENT_ID32   ClientId;                        /* 0x020 */
    uint32_t      ActiveRpcHandle;                 /* 0x028 */
    uint32_t      ThreadLocalStoragePointer;       /* 0x02C */
    uint32_t      ProcessEnvironmentBlock;         /* 0x030 */
    uint32_t      LastErrorValue;                  /* 0x034 */
    uint32_t      CountOfOwnedCriticalSections;    /* 0x038 */
    uint32_t      CsrClientThread;                 /* 0x03C */
    uint32_t      Win32ThreadInfo;                 /* 0x040 */
    uint32_t      User32Reserved[26];              /* 0x044 */
    uint32_t      UserReserved[5];                 /* 0x0AC */
    uint32_t      WOW32Reserved;                   /* 0x0C0 */
    uint32_t      CurrentLocale;                   /* 0x0C4 */
    uint32_t      FpSoftwareStatusRegister;        /* 0x0C8 */
    uint32_t      ReservedForDebuggerInstrumentation[16]; /* 0x0CC */
    uint32_t      SystemReserved1[21];             /* 0x10C */
    uint32_t      HeapFlsData;                     /* 0x160 */
    uint32_t      RngState[4];                     /* 0x164 */
    uint8_t       PlaceholderCompatibilityMode;    /* 0x174 */
    uint8_t       PlaceholderHydrationAlwaysExplicit; /* 0x175 */
    uint8_t       PlaceholderReserved[10];         /* 0x176 */
    uint32_t      ProxiedProcessId;                /* 0x180 */
    uint8_t       ActivationStack[0x18];           /* 0x184 _ACTIVATION_CONTEXT_STACK */
    uint8_t       WorkingOnBehalfTicket[8];        /* 0x19C */
    int32_t       ExceptionCode;                   /* 0x1A4 */
    uint32_t      ActivationContextStackPointer;   /* 0x1A8 */
    uint32_t      InstrumentationCallbackSp;       /* 0x1AC */
    uint32_t      InstrumentationCallbackPreviousPc; /* 0x1B0 */
    uint32_t      InstrumentationCallbackPreviousSp; /* 0x1B4 */
    uint8_t       InstrumentationCallbackDisabled; /* 0x1B8 */
    uint8_t       SpareBytes[0x17];                /* 0x1B9 */
    uint32_t      TxFsContext;                     /* 0x1D0 */
    uint8_t       GdiTebBatch[0x4E0];              /* 0x1D4 _GDI_TEB_BATCH */
    CLIENT_ID32   RealClientId;                    /* 0x6B4 */
    uint32_t      GdiCachedProcessHandle;          /* 0x6BC */
    uint32_t      GdiClientPID;                    /* 0x6C0 */
    uint32_t      GdiClientTID;                    /* 0x6C4 */
    uint32_t      GdiThreadLocalInfo;              /* 0x6C8 */
    uint32_t      Win32ClientInfo[62];             /* 0x6CC */
    uint32_t      glDispatchTable[233];            /* 0x7C4 */
    uint32_t      glReserved1[29];                 /* 0xB68 */
    uint32_t      glReserved2;                     /* 0xBDC */
    uint32_t      glSectionInfo;                   /* 0xBE0 */
    uint32_t      glSection;                       /* 0xBE4 */
    uint32_t      glTable;                         /* 0xBE8 */
    uint32_t      glCurrentRC;                     /* 0xBEC */
    uint32_t      glContext;                       /* 0xBF0 */
    uint32_t      LastStatusValue;                 /* 0xBF4 */
    UNICODE_STRING32 StaticUnicodeString;          /* 0xBF8 */
    uint16_t      StaticUnicodeBuffer[261];        /* 0xC00 */
    uint8_t       Padding0[2];                     /* 0xE0A */
    uint32_t      DeallocationStack;               /* 0xE0C */
    uint32_t      TlsSlots[64];                    /* 0xE10 */
    LIST_ENTRY32  TlsLinks;                        /* 0xF10 */
    uint32_t      Vdm;                             /* 0xF18 */
    uint32_t      ReservedForNtRpc;                /* 0xF1C */
    uint32_t      DbgSsReserved[2];                /* 0xF20 */
    uint32_t      HardErrorMode;                   /* 0xF28 */
    uint32_t      Instrumentation[9];              /* 0xF2C */
    uint8_t       ActivityId[0x10];                /* 0xF50 _GUID */
    uint32_t      SubProcessTag;                   /* 0xF60 */
    uint32_t      PerflibData;                     /* 0xF64 */
    uint32_t      EtwTraceData;                    /* 0xF68 */
    uint32_t      WinSockData;                     /* 0xF6C */
    uint32_t      GdiBatchCount;                   /* 0xF70 */
    uint32_t      CurrentIdealProcessor;           /* 0xF74 */
    uint32_t      GuaranteedStackBytes;            /* 0xF78 */
    uint32_t      ReservedForPerf;                 /* 0xF7C */
    uint32_t      ReservedForOle;                  /* 0xF80 */
    uint32_t      WaitingOnLoaderLock;             /* 0xF84 */
    uint32_t      SavedPriorityState;              /* 0xF88 */
    uint32_t      ReservedForCodeCoverage;         /* 0xF8C */
    uint32_t      ThreadPoolData;                  /* 0xF90 */
    uint32_t      TlsExpansionSlots;               /* 0xF94 */
    uint32_t      MuiGeneration;                   /* 0xF98 */
    uint32_t      IsImpersonating;                 /* 0xF9C */
    uint32_t      NlsCache;                        /* 0xFA0 */
    uint32_t      pShimData;                       /* 0xFA4 */
    uint32_t      HeapData;                        /* 0xFA8 */
    uint32_t      CurrentTransactionHandle;        /* 0xFAC */
    uint32_t      ActiveFrame;                     /* 0xFB0 */
    uint32_t      FlsData;                         /* 0xFB4 */
    uint32_t      PreferredLanguages;              /* 0xFB8 */
    uint32_t      UserPrefLanguages;               /* 0xFBC */
    uint32_t      MergedPrefLanguages;             /* 0xFC0 */
    uint32_t      MuiImpersonation;                /* 0xFC4 */
    uint16_t      CrossTebFlags;                   /* 0xFC8 */
    uint16_t      SameTebFlags;                    /* 0xFCA */
    uint32_t      TxnScopeEnterCallback;           /* 0xFCC */
    uint32_t      TxnScopeExitCallback;            /* 0xFD0 */
    uint32_t      TxnScopeContext;                 /* 0xFD4 */
    uint32_t      LockCount;                       /* 0xFD8 */
    int32_t       WowTebOffset;                    /* 0xFDC */
    uint32_t      ResourceRetValue;                /* 0xFE0 */
    uint32_t      ReservedForWdf;                  /* 0xFE4 */
    uint64_t      ReservedForCrt;                  /* 0xFE8 */
    uint8_t       EffectiveContainerId[0x10];      /* 0xFF0 _GUID */
    uint64_t      LastSleepCounter;                /* 0x1000 */
    uint32_t      SpinCallCount;                   /* 0x1008 */
    uint8_t       Padding1[4];                     /* 0x100C */
    uint64_t      ExtendedFeatureDisableMask;      /* 0x1010 */
    uint32_t      SchedulerSharedDataSlot;         /* 0x1018 */
    uint32_t      HeapWalkContext;                 /* 0x101C */
    uint8_t       PrimaryGroupAffinity[0x0C];      /* 0x1020 _GROUP_AFFINITY */
    uint32_t      Rcu[2];                          /* 0x102C */
} TEB32;
```
可见结构体的成员多且复杂， 这里只需要了解两个成员：
```
NT_TIB32      NtTib;                           /* 0x000 */
uint32_t      ProcessEnvironmentBlock;         /* 0x030 */
```
其中开头的 NtTib 也是一个结构体，定义如下：
```
typedef struct _NT_TIB32 {                    /* sizeof = 0x1C */
    uint32_t ExceptionList;                   /* 0x00 SEH 链头 */
    uint32_t StackBase;                       /* 0x04  */
    uint32_t StackLimit;                      /* 0x08  */
    uint32_t SubSystemTib;                    /* 0x0C  */
    union {
        uint32_t FiberData;                   /* 0x10 */
        uint32_t Version;                     /* 0x10 */
    };
    uint32_t ArbitraryUserPointer;            /* 0x14 */
    uint32_t Self;                            /* 0x18 ← fs:[0x18] */
} NT_TIB32;
```
ExceptionList：指向 \_EXCEPTION_REGISTRATION_RECORD 结构体组成的链表，它用于Windows OS的SEH
Self：\_NT_TIB 结构体的自引用指针，也是TEB结构体的指针
下面通过一段c代码来尝试访问TEB结构体
```C
#include <stdio.h>
#include <windows.h>
#include <winternl.h> // 包含 _TEB 结构体定义

int main() {
    HMODULE hNtdll = LoadLibraryA("ntdll.dll");
    if (hNtdll == NULL) {
        printf("LoadLibrary failed, error: %lu\n", GetLastError());
        return 1;
    }
    
    // 32位 NtCurrentTeb 是导出函数，使用 __stdcall 调用约定
    TEB* (__stdcall *lpNtCurrentTeb)(void) =
        (TEB* (__stdcall *)(void))GetProcAddress(hNtdll, "NtCurrentTeb");
        
    if (lpNtCurrentTeb == NULL) {
        printf("GetProcAddress failed. This should NOT happen on 32-bit.\n");
        FreeLibrary(hNtdll);
        return 1;
    }
    
    TEB* pTEB = lpNtCurrentTeb();
    printf("TEB addr (via GetProcAddress): %p\n", pTEB);
    fflush(stdout);
    
    FreeLibrary(hNtdll);
    getchar();
    return 0;
}
```
`ntdll.dll` 是 Windows 底层的核心动态链接库，包含了大量的原生 API，这里我们需要用到它导出表的`NtCurrentTeb`函数
用 MinGW-w64 i686 将该源码编译成PE32，运行之后打印
TEB addr (via GetProcAddress): 008D6000
发现实际并无啥用，然后可以在x32dbg中调试
![](x32dbg.png)
可以看到，NtCurrentTeb()函数的内部代码非常简单，只返回 FS:[0x18] 的值
其中，FS是段寄存器，(x64的话对应的是GS段寄存器)间接指向TEB地址，于是我们得到
```
FS:[0x18] = TEB起始地址
```
再整理一下，结合FS:0也指向TEB开头，可以理解以下公式
```
FS:[0x18] = TEB.NtTib.Self = address of TIB = address of TEB = FS:0
```
话再说回来前面没有提到的重要成员之二：ProcessEnvironmentBlock，这个就是该TEB对应线程所属的进程的PEB（似乎有点绕口），又可以发现
```
FS:[0x30] = TEB.ProcessEnvironmentBlock = address of PEB
```
### PEB（Process Environment Block，进程环境块）
PEB是存放进程信息的结构体，包含镜像基址、模块链表、命令行、环境变量、堆、是否被调试等信息
在MSDN上对PEB的描述如下：
```
typedef struct _PEB {
  BYTE                          Reserved1[2];
  BYTE                          BeingDebugged;
  BYTE                          Reserved2[1];
  PVOID                         Reserved3[2];
  PPEB_LDR_DATA                 Ldr;
  PRTL_USER_PROCESS_PARAMETERS  ProcessParameters;
  PVOID                         Reserved4[3];
  PVOID                         AtlThunkSListPtr;
  PVOID                         Reserved5;
  ULONG                         Reserved6;
  PVOID                         Reserved7;
  ULONG                         Reserved8;
  ULONG                         AtlThunkSListPtr32;
  PVOID                         Reserved9[45];
  BYTE                          Reserved10[96];
  PPS_POST_PROCESS_INIT_ROUTINE PostProcessInitRoutine;
  BYTE                          Reserved11[128];
  PVOID                         Reserved12[1];
  ULONG                         SessionId;
} PEB, *PPEB;
```
和TEB一样，又是一堆保留字段。。。
下面贴出PEB32的部分结构
```
typedef struct _PEB32 {
    uint8_t       InheritedAddressSpace;           /* 0x00 */
    uint8_t       ReadImageFileExecOptions;        /* 0x01 */
    uint8_t       BeingDebugged;                   /* 0x02 */
    uint8_t       BitField;                        /* 0x03 */
    uint32_t      Mutant;                          /* 0x04 */
    uint32_t      ImageBaseAddress;                /* 0x08 */
    uint32_t      Ldr;                             /* 0x0C */
    uint32_t      ProcessParameters;               /* 0x10 */
    uint32_t      SubSystemData;                   /* 0x14 */
    uint32_t      ProcessHeap;                     /* 0x18 */
    uint32_t      FastPebLock;                     /* 0x1C */
    uint32_t      AtlThunkSListPtr;                /* 0x20 */
    uint32_t      IFEOKey;                         /* 0x24 */
    uint32_t      CrossProcessFlags;               /* 0x28 */
    uint32_t      KernelCallbackTable;             /* 0x2C */
    uint32_t      SystemReserved;                  /* 0x30 */
    uint32_t      AtlThunkSListPtr32;              /* 0x34 */
    uint32_t      ApiSetMap;                       /* 0x38 */
    uint32_t      TlsExpansionCounter;             /* 0x3C */
    uint32_t      TlsBitmap;                       /* 0x40 */
    uint32_t      TlsBitmapBits[2];                /* 0x44 */
    uint32_t      ReadOnlySharedMemoryBase;        /* 0x4C */
    uint32_t      SharedData;                      /* 0x50 */
    uint32_t      ReadOnlyStaticServerData;        /* 0x54 */
    uint32_t      AnsiCodePageData;                /* 0x58 */
    uint32_t      OemCodePageData;                 /* 0x5C */
    uint32_t      UnicodeCaseTableData;            /* 0x60 */
    uint32_t      NumberOfProcessors;              /* 0x64 */
    uint32_t      NtGlobalFlag;                    /* 0x68  ← 经典反调试 */
    uint32_t      Unused;                          /* 0x6C */
    int64_t       CriticalSectionTimeout;          /* 0x70 */
    uint32_t      HeapSegmentReserve;              /* 0x78 */
    uint32_t      HeapSegmentCommit;               /* 0x7C */
    uint32_t      HeapDeCommitTotalFreeThreshold;  /* 0x80 */
    uint32_t      HeapDeCommitFreeBlockThreshold;  /* 0x84 */
    uint32_t      NumberOfHeaps;                   /* 0x88 */
    uint32_t      MaximumNumberOfHeaps;            /* 0x8C */
    uint32_t      ProcessHeaps;                    /* 0x90 */
	…………(未完待续)
} PEB32;
```
我们主要研究以下结构体成员：
```
BeingDebugged;                   /* 0x02 */
ImageBaseAddress;                /* 0x08 */
Ldr;                             /* 0x0C */
ProcessHeap;                     /* 0x18 */
NtGlobalFlag;                    /* 0x68 */
```
#### BeingDebugged：
在最近的windows大版本中（win7之后？），KernelBase.dll 中（之前是在Kernel32.dll）有一个非常重要的api函数
```
BOOL WINAPI IsDebuggerPresent(void);
```
顾名思义，该函数用来判断当前进程是否处于调试状态，判断根据就是读取PEB中的BeingDebugged这个值（**是，则返回1；否，则返回0**），该函数在ida中反汇编如下（此处为x64环境）
![](Pasted_image_20261101135047.png)
这个就是最简单的一种反调试方式，但是很多调试器都会把 PEB->BeingDebugged 的值修改为0，这样就可以轻松绕过该反调试
#### ImageBaseAddress:
该成员指向主模块（通常是.exe文件）在内存中的实际加载基址
我们要访问这个成员，需要
```
HMODULE __stdcall GetModuleHandleW(LPCWSTR SourceString)
```
这个windows api来访问它，函数反编译如下
![[Pasted image 20261101141037.png]]
#### Ldr
这是一个指向 \_PEB_LDR_DATA 结构体的指针，该结构体的大致内容如下
```
typedef struct _PEB_LDR_DATA32 {
    uint32_t      Length;                          /* 0x00 */
    uint8_t       Initialized;                     /* 0x04 */
    uint8_t       Padding0[3];                     /* 0x05 */
    uint32_t      SsHandle;                        /* 0x08 */
    LIST_ENTRY32  InLoadOrderModuleList;           /* 0x0C */
    LIST_ENTRY32  InMemoryOrderModuleList;         /* 0x14 */
    LIST_ENTRY32  InInitializationOrderModuleList; /* 0x1C */
    uint32_t      EntryInProgress;                 /* 0x24 */
    uint8_t       ShutdownInProgress;              /* 0x28 */
    uint8_t       Padding1[3];                     /* 0x29 */
    uint32_t      ShutdownThreadId;                /* 0x2C */
} PEB_LDR_DATA32;
```
这里面有三个 \_LIST\_ENTRY 类型的成员，这个结构体长这样：
```
typedef struct _LIST_ENTRY {
	struct _LIST_ENTRY *Flink;
	struct _LIST_ENTRY *Blink;
}   _LIST_ENTRY, *PLIST_ENTRY;
```
这个大概是提供了一个双向链表的机制，查阅了一下，上面三条双向链表的内容：
- `InLoadOrderModuleList`含义：**模块被加载的顺序**。主 EXE 排第一，之后是按依赖关系加载的 DLL。
- `InMemoryOrderModuleList`含义：**模块在虚拟内存中的地址顺序**（基址从低到高）
- `InInitializationOrderModuleList`含义：**模块 `DllMain` 被调用的顺序**。
####  ProcessHeap
指针，指向进程的默认堆，这个堆的头部（\_HEAP）结构体中，有两个字段 `Flags` 和 `ForceFlags` 和反调试相关
####  NtGlobalFlag
这个`DWORD`的字段是更加直接的一个调试标志位，在进程正常运行时，该值通常为`0`，但是如果进程由调试器创建的话，它的值会被设置为`0x70`（常见典型值）