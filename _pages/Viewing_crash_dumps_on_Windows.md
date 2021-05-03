---
autogenerated: true
title: Viewing crash dumps on Windows
layout: page
---

### Viewing Java Virtual Machine crash logs

If you are running Micro-Manager in the normal way (with the GUI), the
Java Virtual Machine will save a crash log if the whole program crashes.
Most of the time, such a crash means that something went wrong in one of
the C++ components of Micro-Manager.

JVM crash log files are named hs\_err\_pid\*.log, with the process id of
the JVM that crashed, and are placed in the Micro-Manager folder.

Although the JVM normally can catch errors that cause the process to
crash (such as access violations (= segmentation faults)), sometimes the
memory corruption (or some other condition) resulting from the error is
severe enough to cause the JVM's crash handler to fail or not be
executed. In that case, the only way to see what happened (save for
running Micro-Manager with a debugger attached) is to have Windows
produce crash dumps.

Note that, when the JVM saves a crash log, Micro-Manager's windows will
just disappear, with no error dialog box (as far as Windows is
concerned, the program terminated of its own will). When the JVM crash
handler fails, you will usually see Windows display a message saying
"Java(TM) Platform SE binary has stopped working".

### Generating Windows application crash dumps

{% include Note text="The crash dumps saved by this method are binary files. You need a Microsoft debugger (e.g. windbg.exe) to view them." %}

Windows crash dumps are disabled by default (to be precise, saving them
locally is disabled by default; there is a mechanism to allow sending
the dumps to Microsoft, but that is not helpful for our purpose). To
**enable** local saving of crash dumps on **Windows Vista, 7, or
later**, download [this file](Media:media/SaveJVMCrashDumps.zip "wikilink"),
extract the .reg file, and open it (administrator permission will be
required), which will add the necessary settings to the Windows
registry. (See [Microsoft's
documentation](http://msdn.microsoft.com/en-us/library/windows/desktop/bb787181.aspx)
for details on this.)

Following this setup, when java.exe or javaw.exe crashes, a crash dump
should be saved at

` %LocalAppData%\CrashDumps\java.exe.*.dmp`

or

` %LocalAppData%\CrashDumps\javaw.exe.*.dmp`

where the location of `%LocalAppData%` can be checked by typing
`echo %LocalAppData%` into the Command Prompt. The App Data directory is
hidden by default; the easiest way to open the CrashDumps folder is to
open a Command Prompt and type

` start %LocalAppData%\CrashDumps`

Note that crash dump files can get quite large (especially if the
application is using a lot of memory).

[This file](Media:media/StopSavingAllCrashDumps.zip "wikilink") will
**disable** the saving of all Windows application crash dumps.

If you are running **Windows XP**, the method described on [this
page](https://help.github.com/articles/getting-a-crash-dump) should
work.