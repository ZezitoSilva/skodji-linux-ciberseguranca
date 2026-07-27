# 🛡️ Sessão 06 — Desafio MiniCTF Defensivo Linux

**Módulo:** Linux e Cibersegurança  
**Programa:** Skodji Digital — Percurso Reskilling  
**Formador:** Péricles Borges  
**Objectivo:** Integração de OA1 a OA5 — Auditoria, Contenção, Remediação e Validação  
**Plataforma:** TryHackMe — Linux Incident Surface  
**Data:** Julho 2026  

---

## 🎯 Cenário

O servidor Ubuntu da empresa fictícia **"Linux Agency"** apresentava indícios de actividade suspeita e configurações severamente inseguras. A missão foi auditar o sistema, conter os danos, aplicar as correcções e documentar toda a intervenção — como numa resposta a um incidente real.

---

## 📋 Metodologia Aplicada

```
Fase 1 — Identificação e Triagem
Fase 2 — Contenção (Firewall UFW)
Fase 3 — Enrijecimento / Remediação e Validação
```

---

## 🔍 FASE 1 — Identificação e Triagem

### 1.1 Análise de Sockets Activos (Rede e Portas)

**Comando executado:**
```bash
ss -tuln
```

**Propósito:** Listar todas as portas TCP e UDP em estado de escuta (LISTEN) para identificar superfícies de exposição desnecessárias.

**Output obtido:**
```
Netid  State    Recv-Q  Send-Q  Local Address:Port   Peer Address:Port
udp    UNCONN   0       0       127.0.0.53%lo:53      0.0.0.0:*
udp    UNCONN   0       0       10.128.190.89%eth0:68 0.0.0.0:*
udp    UNCONN   0       0       0.0.0.0:631           0.0.0.0:*
udp    UNCONN   0       0       0.0.0.0:5353          0.0.0.0:*
tcp    LISTEN   0       5       127.0.0.1:5901        0.0.0.0:*
tcp    LISTEN   0       128     0.0.0.0:22            0.0.0.0:*
tcp    LISTEN   0       100     0.0.0.0:80            0.0.0.0:*
tcp    LISTEN   0       5       127.0.0.1:631         0.0.0.0:*
tcp    LISTEN   0       128     [::]:22               [::]:*
tcp    LISTEN   0       5       [::1]:5901            [::]:*
```
<img width="589" height="355" alt="image" src="https://github.com/user-attachments/assets/10cb7cd5-2a5a-4400-b0d5-0a24855072fa" />

**✅ Resultado:** Portas activas identificadas: **22 (SSH)**, **80 (HTTP)** e **5901 (VNC)**.

> **Nota:** O comando `nmap -sV localhost` falhou com "command not found". O Nmap não estava instalado e o ambiente de laboratório não tinha conectividade com a internet, inviabilizando a instalação via apt. A análise de portas foi concluída exclusivamente via `ss -tuln`.

<img width="589" height="124" alt="image" src="https://github.com/user-attachments/assets/6f9497e5-5c20-4e16-b0e9-4aa209516cc7" />

---

### 1.2 Auditoria de Contas de Utilizadores

**Comando executado:**
```bash
sudo cat /etc/shadow | awk -F: '($2==""){print $1}'
```
<img width="589" height="22" alt="image" src="https://github.com/user-attachments/assets/27278163-18f0-4592-90ff-b94003fa6b07" />

**Propósito:** Vasculhar o ficheiro criptográfico `/etc/shadow` à procura de utilizadores locais criados sem palavra-passe associada.

**Output obtido:**
```
(sem output — resultado vazio)
```

**✅ Resultado:** Todas as contas activas no sistema exigem autenticação ou estão devidamente bloqueadas (marcadas com `!` ou `*`), eliminando o risco imediato de acessos sem autenticação.

---

### 1.3 Análise de Chaves de Persistência (Authorized Keys)

**Comando executado:**
```bash
cat ~/.ssh/authorized_keys
```

**Propósito:** Auditar as chaves públicas autorizadas na conta para identificar privilégios excessivos ou vectores de persistência deixados por atacantes.

**Output obtido:** Foram identificadas **4 chaves públicas** no ficheiro:

| Identificador | Classificação | Acção Recomendada |
|---|---|---|
| `amiOpenVPN` | ⚠️ **Suspeita** — chave residual de imagem AWS ou persistência maliciosa | Revogar e investigar |
| `cmnatic` | ✅ **Legítima** — conta do criador oficial da sala TryHackMe | Manter |
| `saqib.shabbir` | ⚠️ **Suspeita** — acesso nominal específico sujeito a auditoria | Revogar e verificar |
| `eu-west-3-vuln-vms` | 🚨 **Crítica** — indicativo de infraestrutura exposta pós-comprometimento | Revogar imediatamente |

<img width="589" height="271" alt="image" src="https://github.com/user-attachments/assets/0ed543e2-bd7f-4f6e-8c66-d3226862ada5" />

**⚠️ Resultado:** 3 das 4 chaves são suspeitas ou críticas. A presença de `eu-west-3-vuln-vms` é um indicador forte de comprometimento prévio da infraestrutura.

---

## 🔒 FASE 2 — Contenção (Firewall UFW)

Para conter possíveis acessos maliciosos sem quebrar o acesso à gestão do laboratório virtual, foram adicionadas regras explícitas para as portas necessárias e activada a firewall com política restritiva de entrada.

**Comandos executados:**

```bash
# Permitir SSH (gestão remota)
sudo ufw allow 22/tcp

# Permitir VNC (interface visual do laboratório)
sudo ufw allow 5901/tcp

# Bloquear todo o tráfego de entrada por omissão
sudo ufw default deny incoming

# Activar a firewall
sudo ufw enable
```

**Outputs obtidos:**
```
Rules updated
Rules updated (v6)

Rules updated
Rules updated (v6)

<img width="318" height="117" alt="image" src="https://github.com/user-attachments/assets/fc877af9-c58d-4156-b4e3-1d3be82de7aa" />

Default incoming policy changed to 'deny'
(be sure to update your rules accordingly)

<img width="319" height="58" alt="image" src="https://github.com/user-attachments/assets/5f8cc7ad-3bde-4960-81be-d6f6bfdab94f" />

Firewall is active and enabled on system startup

<img width="321" height="39" alt="image" src="https://github.com/user-attachments/assets/a524e943-00e2-4cd3-ba4f-e1ef5d362386" />

```

**✅ Resultado:** Firewall UFW activada com sucesso. Apenas as portas 22 (SSH) e 5901 (VNC) permanecem acessíveis. Todo o restante tráfego de entrada é bloqueado.

| Porta | Protocolo | Acção | Justificação |
|---|---|---|---|
| 22 | TCP | ALLOW | Gestão remota SSH — essencial |
| 5901 | TCP | ALLOW | Interface VNC do laboratório |
| Todas as outras | — | DENY | Princípio Secure by Default |

---

## 🔧 FASE 3 — Enrijecimento / Remediação e Validação

### 3.1 Endurecimento do Serviço SSH

**Comando executado:**
```bash
sudo nano /etc/ssh/sshd_config
```

**Alterações aplicadas:**

```
# Linha 11 — Bloqueio total de login como root
# ANTES:  #PermitRootLogin prohibit-password
# DEPOIS: PermitRootLogin no

# Desactivação de autenticação por password
# ANTES:  #PasswordAuthentication yes
# DEPOIS: PasswordAuthentication no

# Obrigatoriedade de autenticação por chave criptográfica
# ANTES:  #PubkeyAuthentication yes
# DEPOIS: PubkeyAuthentication yes
```

| Diretiva | Valor Aplicado | Justificação |
|---|---|---|
| `PermitRootLogin` | `no` | Bloqueia login directo como root — elimina vector de escalada de privilégios |
| `PasswordAuthentication` | `no` | Elimina ataques de força bruta — sem password não há o que adivinhar |
| `PubkeyAuthentication` | `yes` | Obriga autenticação exclusiva por par de chaves criptográficas |

<img width="589" height="381" alt="image" src="https://github.com/user-attachments/assets/22ad96b3-0660-4f4a-9395-83e3c982f931" />

<img width="589" height="66" alt="image" src="https://github.com/user-attachments/assets/8e038504-2a75-4cea-a413-53588a7b4cad" />

<img width="589" height="67" alt="image" src="https://github.com/user-attachments/assets/79152f3c-e435-40d6-bb9f-785908035d6d" />

**Reinicialização do serviço:**
```bash
sudo systemctl restart sshd

<img width="589" height="17" alt="image" src="https://github.com/user-attachments/assets/c1c32d59-ddb4-41b9-90bd-a59b20571405" />
```

```
(sem output — reinicialização concluída sem erros)
```

**✅ Resultado:** Serviço SSH reiniciado com sucesso, aplicando imediatamente as directrizes de endurecimento.

---

### 3.2 Tentativa de Aplicação de Patches de Segurança

**Comandos testados:**
```bash
sudo apt update && sudo apt upgrade -y
```
```
0% [Connecting to eu-west-3.ec2.archive.ubuntu.com (2a05:d012:53e:ae02:63d9:eb8...)]
```

<img width="589" height="30" alt="image" src="https://github.com/user-attachments/assets/0c7b6d86-7c1f-45c4-80c3-f8096345cf4e" />

```bash
ping -c 3 8.8.8.8
```
```
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.

--- 8.8.8.8 ping statistics ---
3 packets transmitted, 0 received, 100% packet loss, time 2053ms

<img width="589" height="85" alt="image" src="https://github.com/user-attachments/assets/726459f1-9cde-4066-9093-83be806cf5a7" />
```

**Conclusão:** O ambiente de laboratório está configurado em **isolamento completo de rede** (sem rotas para a internet). A actualização de pacotes foi impossibilitada por este factor. O retorno de 100% de packet loss no ping confirma a ausência de conectividade externa.

---

### 3.3 Tentativa de Auditoria com Lynis

```bash
sudo lynis audit system
```
```
sudo: lynis: command not found
<img width="589" height="31" alt="image" src="https://github.com/user-attachments/assets/70b75ead-6026-42ac-aed0-99ee7edca4ab" />
```

**Conclusão:** O Lynis não estava instalado e não foi possível instalar por ausência de conectividade. A validação da melhoria da postura de segurança foi realizada pelas checagens manuais documentadas nas fases anteriores.

---

## 📊 Resumo da Intervenção

| Fase | Acção | Estado |
|---|---|---|
| **Triagem** | Portas activas identificadas: 22, 80, 5901 | ✅ Concluído |
| **Triagem** | Contas sem password: nenhuma encontrada | ✅ Concluído |
| **Triagem** | 3 chaves SSH suspeitas identificadas em authorized_keys | ⚠️ Requer revogação |
| **Contenção** | UFW activada: deny all incoming, allow 22+5901 | ✅ Concluído |
| **Remediação** | PermitRootLogin → no | ✅ Concluído |
| **Remediação** | PasswordAuthentication → no | ✅ Concluído |
| **Remediação** | PubkeyAuthentication → yes | ✅ Concluído |
| **Remediação** | Patches de segurança — sem conectividade | ❌ Impossibilitado |
| **Validação** | Lynis — sem conectividade para instalação | ❌ Impossibilitado |

---

## 📄 Ficheiros de Configuração Corrigidos

### sshd_config (extracto limpo)

```
# /etc/ssh/sshd_config — Configuração Segura
# Skodji Digital — Lab 6 — Zezito Andrade Silva

Port 22
AddressFamily any
ListenAddress 0.0.0.0
ListenAddress ::

# === SEGURANÇA DE AUTENTICAÇÃO ===
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys

# Desactivar métodos inseguros
KerberosAuthentication no
GSSAPIAuthentication no

Subsystem sftp /usr/lib/openssh/sftp-server
```

### ufw-rules.txt (ufw status verbose)

```
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
5901/tcp                   ALLOW IN    Anywhere
22/tcp (v6)                ALLOW IN    Anywhere (v6)
5901/tcp (v6)              ALLOW IN    Anywhere (v6)
```

---

## 🎯 Acções Pendentes (Pós-Lab)

Em ambiente de produção real, as seguintes acções deveriam ser executadas após restaurar conectividade:

1. **Revogar as 3 chaves SSH suspeitas** de `authorized_keys` (`amiOpenVPN`, `saqib.shabbir`, `eu-west-3-vuln-vms`)
2. **Aplicar patches** com `sudo apt update && sudo apt upgrade -y`
3. **Instalar e executar Lynis** para obter o Hardening Index pós-remediação
4. **Instalar fail2ban** para bloquear automaticamente IPs com múltiplas falhas SSH
5. **Instalar libpam-pwquality** para impor política de passwords fortes
6. **Desactivar a porta 80** (HTTP) se não for necessária para o negócio

---

## 🔗 Referências

- [TryHackMe — Linux Incident Surface](https://tryhackme.com/room/linuxincidentsurface)
- [TryHackMe — Linux Agency](https://tryhackme.com/room/linuxagency)
- [UFW Documentation — Ubuntu](https://help.ubuntu.com/community/UFW)
- [OpenSSH sshd_config Manual](https://man.openbsd.org/sshd_config)
- [CIS Ubuntu 24.04 Benchmark](https://www.cisecurity.org/benchmark/ubuntu_linux)

---


