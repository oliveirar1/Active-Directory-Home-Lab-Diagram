# Active Directory Home Lab Diagram
Domain Controller &amp; Client Network
<p>
  <img src="https://i.imgur.com/V5Ws8OK.png" height="50%" width="50%" alt="Disk Sanitization Steps"/>
</p>

# 🏠 Home Lab: Active Directory + DHCP + DNS + NAT

Este projeto configura um laboratório caseiro para aprender e testar o Active Directory com Windows Server 2019 e um cliente Windows 10 em uma rede virtual interna.

---

## 📌 Objetivos

- Criar um controlador de domínio (DC)
- Configurar DNS, DHCP e NAT no mesmo servidor
- Ingressar cliente Windows 10 no domínio
- Criar +1.000 usuários via PowerShell

---

## 🧱 Requisitos

- [VirtualBox](https://www.virtualbox.org/)
- ISOs:
  - Windows Server 2019
  - Windows 10
- 8 GB RAM ou mais
- 50 GB de espaço livre

---

## 🧩 Arquitetura da Rede

![Diagrama da Rede](./diagram/home-lab-ad.png)

### 🌐 Redes

| Rede       | Tipo      | Subnet         | DHCP         |
|------------|-----------|----------------|--------------|
| Internet   | NAT/Bridge| (via roteador) | automático   |
| Interna AD | Interna   | 172.16.0.0/24  | Servidor DC  |

---

## 🖥️ Configuração das Máquinas

### 🔹 VM 1 - Windows Server 2019

| Configuração | Valor |
|--------------|-------|
| CPU          | 2     |
| RAM          | 4 GB  |
| NIC 1        | NAT ou Bridge (Internet) |
| NIC 2        | Interna (InternalNetwork) |

#### Funções Instaladas
- Active Directory Domain Services
- DNS Server
- DHCP Server
- Routing and Remote Access (RRAS)

#### Endereçamento Interno (NIC 2)
- IP: `172.16.0.1`
- Máscara: `255.255.255.0`
- Gateway: *vazio*
- DNS: `127.0.0.1`

#### DHCP Scope
- Range: `172.16.0.100 - 172.16.0.200`
- Gateway: `172.16.0.1`
- DNS: `172.16.0.1`

---

### 🔹 VM 2 - Windows 10

| Configuração | Valor |
|--------------|-------|
| CPU          | 2     |
| RAM          | 2 GB  |
| NIC          | Interna (InternalNetwork) |
| DHCP         | Automático (servidor DC) |

Após configurar a rede, ingressar no domínio: `lab-ronaldo.com`.

---

## ⚙️ Passos para Configuração

### 1. Criar VMs no VirtualBox
Configure ambas com NICs conforme diagrama.

### 2. Instalar Windows Server 2019 e Windows 10
Configure hostname, atualizações e crie snapshots.

### 3. Configurar o Servidor como DC
```powershell
Install-WindowsFeature AD-Domain-Services, DNS, DHCP, RemoteAccess -IncludeManagementTools.

---

### 4. Promover o DC

Install-ADDSForest -DomainName "lab-ronaldo.com"

### 5. Configurar DHCP
Scope: 172.16.0.100-200

DNS: 172.16.0.1

Gateway: 172.16.0.1

### 6. Configurar NAT com RRAS
Use o assistente do "Routing and Remote Access" para ativar NAT da NIC externa para a interna.

7. Ingressar Cliente no Domínio
No Windows 10:

Sistema > Nome do Computador > Alterar > Ingressar domínio: lab-ronaldo.com

---

### 🧪 Script PowerShell: Criar +1000 Usuários

for ($i=1; $i -le 1000; $i++) {
    $username = "user$i"
    New-ADUser -Name $username -SamAccountName $username -AccountPassword (ConvertTo-SecureString "P@ssword123" -AsPlainText -Force) -Enabled $true
}

### 💾 Salvar como create-users.ps1 e executar no PowerShell com permissões administrativas.

### 📘 Referências
Documentação Microsoft AD DS
VirtualBox Manual
PowerShell AD Module

### 🔐 Segurança
Este ambiente não é seguro para produção. Use apenas para fins educacionais em redes isoladas.
