---
layout: post
title:  "使服务程序像模像样"
date:   2008-12-23 16:54:00
categories: desktop service
tags: windows service
---

* content
{:toc}

写windows的服务程序当然算不上什么高级编程，但是一般人可能不太注意服务程序的形象问题。打开服务管理器，服务名称就是程序名，更没有描述。看起来挺别扭。起始要使服务好看一点，只需要几步即可（以VC6为例）：

1. 将原来Install函数的CreateService调用改为
```cpp
SC_HANDLE hService = ::CreateService(
    hSCM,                       // SC_HANDLE hSCManager,        handle to service control manager database
    m_szServiceName,            // LPCTSTR   lpServiceName,     pointer to name of service to start
    _T("世纪德润看护服务"),     // LPCTSTR   lpDisplayName,     pointer to display name
    SERVICE_ALL_ACCESS,         // DWORD     dwDesiredAccess,   type of access to service
    SERVICE_WIN32_OWN_PROCESS,  // DWORD     dwServiceType,     type of service
    SERVICE_AUTO_START,         // DWORD     dwStartType,       when to start service
    SERVICE_ERROR_NORMAL,       // DWORD     dwErrorControl,    severity if service fails to start
    szFilePath,                 // LPCTSTR   lpBinaryPathName,  pointer to name of binary file
    NULL,                       // LPCTSTR   lpLoadOrderGroup,  pointer to name of load ordering group
    NULL,                       // LPDWORD   lpdwTagId,         pointer to variable to get tag identifier
    _T("RPCSS/0"),              // LPCTSTR   lpDependencies,    pointer to array of dependency names
    NULL,                       // LPCTSTR   lpServiceStartName,pointer to account name of service
    NULL);                      // LPCTSTR   lpPassword         pointer to password for service account
```

在::CloseServiceHandle(hSCM);以前添加：
```cpp
// Need to acquire database lock before reconfiguring.
SC_LOCK sclLock = LockServiceDatabase(hSCM);
if (sclLock != NULL)
{
// Open a handle to the service.
SC_HANDLE hService = OpenService(
    hSCM,                   // SCManager database
    m_szServiceName,        // name of service
    SERVICE_CHANGE_CONFIG); // need CHANGE access

    if (hService != NULL)
    {
        SERVICE_DESCRIPTION sdBuf;
        sdBuf.lpDescription = _T("提供北京世纪德润科技有限公司服务程序的看护服务。");
        if (ChangeServiceConfig2 (hService, SERVICE_CONFIG_DESCRIPTION, &sdBuf))
        {
            // MessageBox(NULL, "Change SUCCESS", "", MB_SERVICE_NOTIFICATION);
        }
        CloseServiceHandle(hService);
    }
    UnlockServiceDatabase(sclLock);
}
```
这就改了服务名称，添加了服务描述。

至于对于VS2003及以上，我喜欢从库里面拎出来相关部分，然后修改，类似于：
```cpp
template <class T, UINT nServiceNameID>
class ATL_NO_VTABLE CDRAtlServiceModuleT : public CAtlExeModuleT<T>
```
