---
layout: post
title: Using Old Windows Symbols with Ghidra in Linux
author: Danny Quist
---

Recently, while developing course material for a reverse engineering course I was making, I needed to get the symbols for the venerable `sol.exe`. Unfortunately the world's greatest solitaire program is no longer shipped with windows, and subsequently Microsoft's symbol servers have stopped providing debug information for it. The last complication was Ghidra's support for PDB is limited to Windows only systems. This guide will walk through how I got the symbols for an unsupported OS (XP) working inside of the Ghidra Linux client.
<!--more-->

![Ghidra's Symbol Tree of sol.exe sans symbols](/assets/2019-07-17-windows-symbols-ghidra-10aba0ec.png)

*Figure 1: A decidedly not-fun listing of functions in the sol.exe*

When trying to load the symbols remotely, which works for other executables, Ghidra has issues downloading the old symbols:

![](/assets/2019-07-17-windows-symbols-ghidra-e4b44240.png)
*Figure 2: Ghidra griping about symbols not being available for Windows XP*

It is understandable that Microsoft wouldn't provide software for this outdated operating system. That did not, however, allow me to look at the sol.exe in
Unfortunately it is not possible to avoid Windows entirely. I haven't had much luck with the Linux based tools to work with PDB files, but luckily Ghidra has a solution. pdb.xml files can be used on Linux for the same purpose.

# 1. Find the Symbols

My lazy web searching did not find a copy of the PDB files readily available through Microsoft's Official sources, but thankfully others have made older symbol files available. I was able to find a copy of `WindowsXP-KB936929-SP3-x86-symbols-full-ENU.exe` on the internet. Given that the file was not from a reputable source, I used a VM to extract and perform the steps.

Ghidra (on non Windows installations) cannot directly read the PDB file format. You have to do some preprocessing of the files to convert them to the `pdb.xml` format. Thankfully Ghidra includes a script to convert `.pdb` files to `.pdb.xml` in `support/createPdbXmlFiles.bat`. Tragically that is a batch file, so we must have a Windows VM.

# 2. Extract the Symbols

Execute the symbols file and have it install normally on a Windows system. The files will be placed into the C:\Windows\Symbols directory. The next step for converting the XML files is non-obvious. Ghidra needs a copy of the msdia140.dll file correctly loaded onto your system. This brings up a few questions: What is msdia140.dll and most importantly, where can I find it? One method is to install Visual Studio community edition, however only newer versions of the DLL were available. Luckily, [@malwaretech has solved all of your problems for you with an archive of the file and instructions to get it working](https://github.com/MalwareTech/MSDIA-x64).

Getting the file installed is straight-forward. Using Powershell running as administrator, copy and register the file

```bash
C:\> xcopy msdia140.dll %systemroot%\system32
C:\> regsvr32 %systemroot%\system32\msdia140.dll
```

The next step is to use the `createPdbXmlFiles.bat` to convert all of the raw PDBs to the xml format. Run the following:

```bash
C:\ghidra\support> createPdbXmlFiles.bat C:\Windows\Symbols

```

You can now copy the C:\Windows\Symbols directory to your Linux machine. Once the copy is complete you can load the XML formatted version of the PDB. Import by going to file->load PDB. Be sure to auto analyze the file again (Analysis->Auto analyze) and you should see a successful file with relevant packages in place:

![](/assets/2019-07-17-windows-symbols-ghidra-92e04d24.png)

*Figure 3: Correct symbols loaded into sol.exe*

You can now explore the file with all the symbol names restored.

# Acknowledgements

Thanks to [@MalwareTech](http://www.malwaretech.com/) for posting a copy of the msdia140.dll files to their website.
