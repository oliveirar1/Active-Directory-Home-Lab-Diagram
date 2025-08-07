# 🏠 Home Lab: Active Directory + DHCP + DNS + NAT

Laboratório virtual completo com Windows Server 2019 e Windows 10 para praticar e testar Active Directory, DNS, DHCP e NAT usando VirtualBox.

---

## 🎯 Objetivos

- 🖥️ Criar um Controlador de Domínio (Domain Controller)
- 🌐 Configurar DNS, DHCP e NAT no servidor
- 💻 Ingressar uma VM cliente Windows 10 no domínio
- 🔄 Criar +1000 usuários automaticamente via PowerShell

---

## 📦 Requisitos

- VirtualBox
- ISOs:
  - Windows Server 2019
  - Windows 10
- 💾 50 GB de espaço livre em disco
- 🧠 8 GB de RAM (mínimo)

---

## 🗺️ Arquitetura de Rede

| Rede       | Tipo      | Subnet         | DHCP         |
|------------|-----------|----------------|--------------|
| Interna AD | Interna   | 172.16.0.0/24  | Servidor DC  |
| Internet   | NAT/Bridge| (via roteador) | Automático   |

> Veja o diagrama abaixo:
![Diagrama da Rede](https://i.imgur.com/V5Ws8OK.png)

---

## 🛠️ Configuração das Máquinas Virtuais

### 🔹 VM 1 - Windows Server 2019 (Domain Controller)

| Recurso     | Valor                         |
|-------------|-------------------------------|
| CPU         | 2 vCPU                        |
| RAM         | 4 GB                          |
| NIC 1       | NAT ou Bridge (Internet)      |
| NIC 2       | Interna (InternalNetwork)     |
| Funções     | AD DS, DNS, DHCP, RRAS        |

#### 🧭 Endereço da NIC Interna
- IP: `172.16.0.1`
- Máscara: `255.255.255.0`
- Gateway: *(vazio)*
- DNS: `127.0.0.1`

#### 🎯 Escopo DHCP
- Range: `172.16.0.100 - 172.16.0.200`
- Gateway: `172.16.0.1`
- DNS: `172.16.0.1`

---

### 🔹 VM 2 - Windows 10 (Cliente)

| Recurso     | Valor                         |
|-------------|-------------------------------|
| CPU         | 2 vCPU                        |
| RAM         | 2 GB                          |
| NIC         | Interna (InternalNetwork)     |
| DHCP        | Automático (via DC)           |

Depois de configurar a rede, a máquina será ingressada no domínio `xxx-ronaldo.com`.

---

## ⚙️ Etapas de Configuração

### 1. Criar VMs no VirtualBox
Configure as duas VMs com as NICs conforme o diagrama.

### 2. Instalar os Sistemas Operacionais
Instale o Windows Server 2019 na VM 1 e o Windows 10 na VM 2.

### 3. Configurar o Servidor como DC

```powershell
Install-WindowsFeature AD-Domain-Services, DNS, DHCP, RemoteAccess -IncludeManagementTools
```

### 4. Promover o DC

```powershell
Install-ADDSForest -DomainName "xxx-ronaldo.com"
```

### 5. Configurar o DHCP

- Range: `172.16.0.100 - 172.16.0.200`
- Gateway: `172.16.0.1`
- DNS: `172.16.0.1`

### 6. Configurar o NAT com RRAS
Use o assistente do **Routing and Remote Access** e ative NAT da NIC externa para a interna.

### 7. Ingressar Cliente no Domínio

No Windows 10:

```text
Sistema > Nome do Computador > Alterar > Domínio: xxx-ronaldo.com
```

---

## 🧪 Script PowerShell - Criar 1000+ Usuários

```powershell
# ----- Edit these Variables for your own Use Case ----- #
$PASSWORD_FOR_USERS   = "Password1"
$USER_FIRST_LAST_LIST = Get-Content .\names.txt
# ------------------------------------------------------ #

$password = ConvertTo-SecureString $PASSWORD_FOR_USERS -AsPlainText -Force
New-ADOrganizationalUnit -Name _USERS -ProtectedFromAccidentalDeletion $false

foreach ($n in $USER_FIRST_LAST_LIST) {
    $first = $n.Split(" ")[0].ToLower()
    $last = $n.Split(" ")[1].ToLower()
    $username = "$($first.Substring(0,1))$($last)".ToLower()
    Write-Host "Creating user: $($username)" -BackgroundColor Black -ForegroundColor Cyan
    
    New-AdUser -AccountPassword $password `
               -GivenName $first `
               -Surname $last `
               -DisplayName $username `
               -Name $username `
               -EmployeeID $username `
               -PasswordNeverExpires $true `
               -Path "ou=_USERS,$(([ADSI]`"").distinguishedName)" `
               -Enabled $true
}
```

Salve como `create-users.ps1` e execute com privilégios de administrador.

---

## 📚 Referências

- [Documentação Microsoft Active Directory](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/)
- [Manual VirtualBox](https://www.virtualbox.org/manual/UserManual.html)
- [PowerShell AD Module](https://learn.microsoft.com/en-us/powershell/module/addsadministration/?view=windowsserver2022-ps)

---

## 🔐 Aviso de Segurança

⚠️ Este ambiente **não é seguro para produção**. Use apenas para fins **educacionais** em rede isolada.



