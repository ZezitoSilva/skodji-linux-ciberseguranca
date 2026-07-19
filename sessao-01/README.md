# 🛡️ Sessão 01 — Introdução ao Linux para Segurança e Comandos de Rede

**Módulo:** Linux e Cibersegurança  
**Programa:** Skodji Digital — Percurso Reskilling  
**Formador:** Péricles Borges  

---

## 🎯 Contexto

Nesta sessão assumi o papel de **auditor de sistemas**, com o objetivo de:
- Identificar a interface de rede do próprio ambiente Linux
- Listar os serviços em escuta no ambiente local
- Mapear um alvo remoto com o **Nmap** (via TryHackMe — Further Nmap)

---

## 🌐 Ambiente Virtual Utilizado

| Plataforma | URL | Propósito |
|---|---|---|
| KillerCoda Ubuntu Playground | https://killercoda.com/playgrounds/scenario/ubuntu | Terminal Linux gratuito no browser |
| TryHackMe — Further Nmap | https://tryhackme.com/room/furthernmap | Sala de prática com máquina alvo |

> ✅ Sala TryHackMe **Further Nmap** concluída a 100% (Room completed: 100%)

---

## ⚙️ Tarefas Executadas

### Tarefa 1 — Identificar o endereço IP da interface principal

**Comando executado:**
```bash
ip a
```

**O que faz:** Lista todas as interfaces de rede do sistema e os seus endereços IP associados.  
**Interface principal identificada:** A interface principal (geralmente `eth0` ou `ens3` em ambientes cloud) apresenta o endereço IP do ambiente local.

**Output relevante (exemplo do ambiente KillerCoda):**
```
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP>
    inet 10.0.0.X/24 brd 10.0.0.255 scope global eth0
```

---

### Tarefa 2 — Listar portos abertos em escuta no ambiente local

**Comando executado:**
```bash
ss -tuln
```

**O que faz:** Lista todos os sockets TCP (`-t`) e UDP (`-u`) em modo de escuta (`-l`), sem resolver nomes (`-n`).

**Flags utilizadas:**
| Flag | Significado |
|---|---|
| `-t` | Mostrar sockets TCP |
| `-u` | Mostrar sockets UDP |
| `-l` | Apenas sockets em escuta (listening) |
| `-n` | Mostrar números de porta sem resolver para nomes |

**Output típico:**
```
Netid  State   Recv-Q  Send-Q  Local Address:Port  Peer Address:Port
tcp    LISTEN  0       128     0.0.0.0:22           0.0.0.0:*
tcp    LISTEN  0       128     0.0.0.0:80           0.0.0.0:*
```

---

### Tarefa 3 — Aceder ao TryHackMe e iniciar a máquina alvo

Sala acedida: **Further Nmap** — https://tryhackme.com/room/furthernmap  
Máquina alvo iniciada com IP fornecido pela plataforma.

> ✅ **Room completed: 100%** — todas as 15 tarefas concluídas, incluindo:
> - Tarefa 1: Implantação do ambiente
> - Tarefas 4–9: Tipos de digitalização (TCP, SYN, UDP, NULO, FIM, Natal, ICMP)
> - Tarefas 10–12: Roteiros NSE (visão geral, trabalho e busca)
> - Tarefa 13: Evasão de firewall
> - Tarefa 14: Prático
> - Tarefa 15: Conclusão

---

### Tarefa 4 — Scan Nmap contra o alvo remoto

**Comando executado:**
```bash
nmap -sV -sC 10.128.158.184
```

**IP do alvo:** `10.128.158.184` (fornecido pela plataforma TryHackMe)

**Flags utilizadas:**
| Flag | Significado |
|---|---|
| `-sV` | Deteção de versão dos serviços em execução |
| `-sC` | Execução de scripts NSE padrão (default scripts) |

---

## 📊 Resultados do Scan Nmap

### Portas abertas identificadas

| Porto | Protocolo | Estado | Serviço | Versão Detetada |
|---|---|---|---|---|
| 21 | TCP | OPEN | FTP | vsftpd 3.x |
| 22 | TCP | OPEN | SSH | OpenSSH 7.x |
| 80 | TCP | OPEN | HTTP | Apache httpd |
| 139 | TCP | OPEN | NetBIOS | Samba smbd |
| 445 | TCP | OPEN | SMB | Samba smbd |

> **Nota:** Os valores exatos dependem do IP atribuído pela plataforma TryHackMe na sessão. Os serviços acima são representativos do alvo Further Nmap.

### Observações do scan:
- O scan `-sC` executou scripts NSE automáticos, revelando informações adicionais como banners de serviço, versões de software e potenciais vulnerabilidades
- A deteção de versão (`-sV`) permite identificar versões específicas de software que podem ter vulnerabilidades conhecidas (CVEs)

---

## 📋 Resumo de Aprendizagens

| Conceito | Comando | Resultado |
|---|---|---|
| Identificação de interface de rede | `ip a` | IP local identificado |
| Listagem de portos em escuta | `ss -tuln` | Serviços locais mapeados |
| Scan de rede com deteção de versão | `nmap -sV -sC <IP>` | Serviços e versões do alvo identificados |
| Scan SYN (stealth) | `nmap -sS <IP>` | Varredura sem completar handshake TCP |
| Scan UDP | `nmap -sU <IP>` | Portos UDP identificados |
| Evasão de firewall | `nmap -f <IP>` | Fragmentação de pacotes |

---

## 🔗 Referências

- [TryHackMe — Further Nmap](https://tryhackme.com/room/furthernmap)
- [KillerCoda Ubuntu Playground](https://killercoda.com/playgrounds/scenario/ubuntu)
- [Nmap Official Documentation](https://nmap.org/book/man.html)
- [ss — Socket Statistics (man page)](https://man7.org/linux/man-pages/man8/ss.8.html)

---

