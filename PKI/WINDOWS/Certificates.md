
> Ivan Ristic - Bulletproof TLS
#### Cryptographic Policies

Windows Server Defaults to TLS 1.2


#### SCHANNEL

*Microsoft Secure Channel* is a cryptographic component that implements a set of protocols designed to enable secure communication. *Schannel* is official SSL/TLS library on Windows platforms. 

#### Microsoft Root Certificate Program

The *Microsoft Root Certificate Program* maintains a collection of certificates trusted in Windows operating systems. Required root certificates are automatically retrieved by Microsoft on the fly.

**Blacklisting Trusted Certificates**

Deleting a trusted root certificate is not enough. The OS will automatically download missing certificates. To blacklist a certificate it must be placed in the *Untrusted Certificates* store.

**Disabling the Auto-Update of Root Certificates**

Local Group Policy Editor

Computer Configuration -> Administrative Template -> System -> Internet Communication Management -> Internet Communication Settings

**Schannel Configuration**
`Computer\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL`

To disable an algorithm, create `DWORD` entry called `Enabled` under the correct key and set its value to 0.

`certutil`

See the TLS protocols

List enabled cipher suites

```powershell
Get-TlsCipherSuite
```

Filter out the names only

```powershell
Get-TlsCipherSuite | ForEach-Object { $_.Name }
```

```
TLS_AES_256_GCM_SHA384
TLS_AES_128_GCM_SHA256
TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384
TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256
TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
TLS_DHE_RSA_WITH_AES_256_GCM_SHA384
```

To specify a cipher suite order. In Local Group Policy Editor

Computer Configuration > Administrative Templates > Network > SSL Configuration Settings.

To configure custom cipher suites

`HKEY_LOCAL_MACHINE\SOFTWARE\Policies\Microsoft\Cryptography\↩ Configuration\SSL\00010002`

Create a multi-string key and add the cipher suites in order of appearance. UNTESTED

To create cipher restrictions

```
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Cryptography\OID\EncodingType 0\CertDllCreateCertificateChainEngine\Config\Default
```

`WeakMd5ThirdPartySha256Allow`

#### Configuring Renegotiation

To disable server-initiated renegotiation

Set to any nonzero value

```
HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\↩ SecurityProviders\SCHANNEL\DisableRenegoOnServer
```

To enforce secure renegotiation (Strict Renegotiation), i.e., accept secure connections only from clients who implement secure negotiation

Set to zero

```
HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\SecurityProviders\SCHANNEL\AllowInsecureRenegoClients
```

To allow insecure renegotiation

```
HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\↩ SecurityProviders\SCHANNEL\AllowInsecureRenegoServers
```
#### Remote Desktop Certificate

NE RABOTI

```powershell
Get-WmiObject -Namespace "Root\CIMv2\TerminalServices" -Class Win32_TSGeneralSetting
```

Time and date sync

```powershell
w32tm /query /status
Get-Date
w32tm /resync
w32tm /config /syncfromflags:domhier /update
net stop w32time
net start w32time
w32tm /resync
w32tm /config /manualpeerlist:"time.windows.com,0x8" /syncfromflags:manual /reliable:YES /update
```

List certificates in the Personal store

```powershell
Get-ChildItem -Path "Cert:\LocalMachine\My"
```

Create a registry key with the certificate thumbprint. reg_binary key.

```powershell
wmic /namespace:\\root\cimv2\TerminalServices PATH Win32_TSGeneralSetting Set SSLCertificateSHA1Hash="6fc82fb0d88a15daa30795c8f7b00977d3277606"
```

Check out the terminal services config

```powershell
Get-WmiObject -Namespace "Root\CIMv2\TerminalServices" -Class Win32_TSGeneralSetting
```

Extract information about certificates

```powershell
Get-ChildItem -Path "Cert:\LocalMachine\My" | Format-List Subject, Thumbprint, HasPrivateKey, EnhancedKeyUsageList
```

Private key of the Server certificate in the Personal Store must have NETWORK SERVICE read permissions.

Right-click on the server certificate -> Manage Private Keys -> Add NETWORK SERVICE

