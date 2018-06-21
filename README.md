# ida-utils

A small but brave and growing collection of advice, links and observations regarding reverse engineering using IDA Pro.

## Reversing COM binaries

### Understanding COM objects/binaries

[COM](https://msdn.microsoft.com/en-us/library/windows/desktop/ms694363(v=vs.85).aspx) - Component Object Model

https://www.codeproject.com/Articles/13601/COM-in-plain-C
- one of the best articles I have read. I highly recommend it

https://reverseengineering.stackexchange.com/questions/13282/ida-pro-list-com-methods
- a very informative thread

http://bytepointer.com/resources/index.htm
- a very interesting site. Highly recommend all the articles regarding COM

http://www.moserware.com/2008/01/finally-understanding-com-after.html
- interesting article with references to other good links for understanding COM

http://www.moserware.com/2009/04/using-obscure-windows-com-apis-in-net.html
- interesting article referenced in the previous recommendation

COM functions reside in ole32.dll `C:\Windows\System32\ole32.dll`

More information regarding COM can be found everywhere since [it is as old as me](https://en.wikipedia.org/wiki/Component_Object_Model).


### COM reversing tools

- [IDA Pro](https://www.hex-rays.com/products/ida/) :smile:
- [Win32 Python COM module](http://timgolden.me.uk/pywin32-docs/pythoncom.html)
- [RCE COM Tools library](http://www.woodmann.com/collaborative/tools/index.php/Category:COM_Tools)
- [Microsoft OLE-COM Object Viewer](https://msdn.microsoft.com/en-us/library/windows/desktop/ms688269(v=vs.85).aspx)
    - the binary comes when the Windows SDK. On my machine, I found the binary as follows (path and sample MD5 hash):

```
C:\Program Files (x86)\Windows Kits\10\bin\10.0.17134.0\arm64\oleview.exe - dd683d280b74d2cc2e6a31a574ac6da0
C:\Program Files (x86)\Windows Kits\10\bin\10.0.17134.0\x64\oleview.exe   - 3cec2bf41e410926f62e189bef547d30
C:\Program Files (x86)\Windows Kits\10\bin\10.0.17134.0\x86\oleview.exe   - 0eeccd530de75c398329a1ba0194614f
```

### Using IDA Pro

#### Types

As ashamed as I am, I must admit I originally did not know in what type library (if any) I could find the IDA structures relating to COM.

First I used IDA's load header feature to load headers such as [guiddef.h](https://github.com/tpn/winsdk-10/blob/master/Include/10.0.10240.0/shared/guiddef.h).
The files are originally found when installing the Windows SDK (in my case there were in `C:\Program Files (x86)\Windows Kits\10\Include\10.0.17134.0`).
Initially I found most of my required headers, online, [here](https://github.com/tpn/winsdk-10/blob/master/Include/10.0.10240.0/) for example.

A second attempt was as to create an IDA .til file. Not knowing all the header files I would need I parsed the 133 functions pages displayed here: [MSDN list of functions that are provided by COM.](https://msdn.microsoft.com/en-us/library/windows/desktop/ms680586(v=vs.85).aspx) to find out that all the functions were exported by:
```
Callobj.h
Combaseapi.h
GuidDef.h
Messagedispatcherapi.h
Objbase.h
Ole2.h
Olectl.h
ROApi.h
Urlmon.h
```
witch I subsequently collected from the SDK in order to build into the .til. At this point I realised the types were probabil in a visual studio type library, something that IDA has.

The type library I was looking for was:
`vc9 - Visual Studio v9 headers (without windows.h)`
The header files are also found in `vc6win - Visual C++` but with a different flavor.

One could have used something similar to `for /R %i in (*.til) do (tilib.exe -lc "%i" | grep GUID -c | grep -v 0)` to find any referenced target structures, but where would the reverse engineering fun in that be?

After loading the type library and doing a type change, such beauty beholds, an example:
![IUnknown](/resources/IUnknown_type.png)

#### Scripts
Haven't found many.

- https://github.com/noobdoesre/py-com-tools

#### Plugins

IDA already comes with:

- [Dieter Spaar's COM Interface Plugin](https://www.hex-rays.com/products/ida/support/download.shtml)

- [Class Informer](https://sourceforge.net/projects/classinformer/) plugin by Sirmabus that can help reconstruct RTTI information for your COM object. It requires IDA Pro 6.9 or greater.

