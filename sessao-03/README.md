# 🛡️ Sessão 03 — Hardening de Redes Linux e Configuração de Firewalls

**Módulo:** Linux e Cibersegurança  
**Programa:** Skodji Digital — Percurso Reskilling  
**Formador:** Péricles Borges  
**Objetivo de Aprendizagem:** OA3 · Aplicar  
**Data de conclusão:** Julho 2026  

---

## 🎯 Contexto

Nesta sessão configurei uma política defensiva estrita para impedir acessos não autorizados a serviços críticos de um servidor Linux, utilizando duas ferramentas complementares:

| Ferramenta | Função |
| :--- | :--- |
| **UFW (Firewall Descomplicado)** | Política padrão e regras de serviço |
| **iptables** | Bloqueio direto de IPs maliciosos a nível de kernel |
| **TryHackMe — Fundamentos de Segurança de Rede** | Análise forense de logs de perímetro (firewall, VPN, WAF) |

---

## 🔍 PARTE 1 — Análise de Logs do Perímetro (Tarefa 6)

Ficheiros analisados em `~/Desktop/Perimeter_logs/task6`:

| Ficheiro | Componente |
| :--- | :--- |
| `firewall_logs.txt` | Registro de tráfego filtrado pelo firewall |
| `waf_logs.txt` | Web Application Firewall — ataques bloqueados |
| `vpn_logs.txt` | Gateway VPN — autenticações falhas e bem sucedidas |

### Questão 1 — Qual IP está fazendo port scan no firewall?

```bash
awk '{print $5}' firewall_logs.txt | awk -F':' '{print $1, $2}' | sort -u | awk '{print $1}' | sort | uniq -c | sort -nr | head -n 5
```

**Saída:**
```text
     47 203.0.113.10
      3 10.0.0.1
```

* **✅ Resposta:** `203.0.113.10`
* **Detalhe:** Volume elevado de esforço para portos diferentes = padrão de port scan.

---

### Questão 2 — Qual IP é responsável pelos ataques WAF bloqueados?

```bash
cat waf_logs.txt | grep -i "BLOCK"
```

**Saída:**
```text
198.51.100.12 - "GET /admin/../etc/passwd" 403 BLOCK SQLi
198.51.100.12 - "POST /login' OR 1=1--" 403 BLOCK SQLi
198.51.100.12 - "GET /../../etc/shadow" 403 BLOCK Traversal
198.51.100.12 - "<script>alert(1)</script>" 403 BLOCK XSS
```

* **✅ Resposta:** `198.51.100.12` — responsável por SQLi, XSS e Directory Traversal.

---

### Questão 3 — Quantas tentativas de força falharam na VPN?

```bash
grep -i "FAILED_AUTH" vpn_logs.txt | wc -l
```

**Saída:**
```text
90
```

* **✅ Resposta:** `90` experiências falhadas.

---

### Questão 4 — Qual IP tentou força bruta contra o gateway VPN?

```bash
grep -i "FAILED_AUTH" vpn_logs.txt | awk '{print $5}' | cut -d':' -f1 | sort | uniq -c | sort -nr | head -n 5
```

**Saída:**
```text
     90 45.137.22.13
```

* **✅ Resposta:** `45.137.22.13` — 90 tentativas falhadas.

---

## 🚨 PARTE 2 — Desafio Prático de Resposta a Incidentes (Challenge)

Ficheiros analisados em `~/Desktop/Perimeter_logs/challenge`:

| Ficheiro | Componente |
| :--- | :--- |
| `firewall.log` | Varreduras e bloqueios |
| `vpn_auth.log` | Autenticações VPN — falhas e sucessos |
| `ids_alerts.log` | Alertas IDS — atividade maliciosa interna |

### Questão 5 — Qual IP externo fez maior reconhecimento?

```bash
cat firewall.log | grep "BLOCK" | awk '{print $5}' | cut -d':' -f1 | sort | uniq -c | sort -nr | head -n 1
```

**Saída:**
```text
    279 203.0.113.45
```

* **✅ Resposta:** `203.0.113.45` — 279 bloqueios registrados.

---

### Questão 6 — Qual host interno foi alvo das varreduras?

```bash
cat firewall.log | grep "BLOCK" | awk '{print $7}' | sort -u | cut -d':' -f1 | sort | uniq -c | sort -nr
```

**Saída:**
```text
    279 10.0.0.20
```

* **✅ Resposta:** `10.0.0.20`

---

### Questão 7 — Qual usuário foi alvo nos logs VPN?

```bash
cat vpn_auth.log | grep "FAIL" | awk '{print $4}' | sort | uniq -c
```

**Saída:**
```text
     90 svc_backup
```

* **✅ Resposta:** `svc_backup` — conta de serviço com 90 falhas.

---

### Questão 8 — Qual IP interno foi atribuído após login VPN bem sucedido?

```bash
cat vpn_auth.log | grep "SUCCESS" | grep -i "svc"
```

**Saída:**
```text
2021-04-20 11:23:45 VPN SUCCESS user=svc_backup src=203.0.113.45 tunnel_ip=10.8.0.23
```

* **✅ Resposta:** `10.8.0.23` — IP de túnel direcionado ao atacante após comprometimento.

---

### Questão 9 — Qual host interno efetuou beacon para o servidor C2?

```bash
grep -i "Trojan" ids_alerts.log | head -n 5
```

**Saída:**
```text
2021-04-20 11:45:12 ALERT src=10.0.0.60 dst=198.51.100.77 sig="ET TROJAN C2 Beaconing"
```

* **✅ Resposta:** `10.0.0.60`

---

### Questão 10 — Qual IP é o servidor C2?

*Identificado na mesma saída da Questão 9.*

* **✅ Resposta:** `198.51.100.77`

---

### Questão 11 — Qual host apresentou exfiltração de dados?

```bash
grep -i "Data" ids_alerts.log | head -n 5
```

**Saída:**
```text
2021-04-20 12:10:33 ALERT src=10.0.0.51 dst=198.51.100.77 sig="HTTP POST Large Upload"
2021-04-20 12:12:45 ALERT src=10.0.0.51 dst=198.51.100.77 sig="ET INFO Data Exfiltration"
```

* **✅ Resposta:** `10.0.0.51`

---

## 📋 Tabela de Indicadores de Comprometimento (IoCs)

| Ameaça | IP Atacante | Destino | Detalhe |
| :--- | :--- | :--- | :--- |
| Varredura de Portos | 203.0.113.10 | Infraestrutura | Reconhecimento de Portos |
| Ataques Web (WAF) | 198.51.100.12 | Servidor Web | SQLi, XSS, Travessia de diretórios |
| Bruta VPN | 45.137.22.13 | VPN de gateway | 90 falhas de suporte |
| Reconhecimento Avançado | 203.0.113.45 | 10.0.0.20 | 279 bloqueios sem firewall |
| Sequestro de Credencial | 203.0.113.45 | Conta: `svc_backup` | Sucesso VPN — túnel `10.8.0.23` |
| Comunicação C2 | 10.0.0.60 | 198.51.100.77 | Sinalização ET TROJAN C2 |
| Exfiltração de Dados | 10.0.0.51 | 198.51.100.77 | Envio de arquivos grandes via HTTP POST |

---

## 🔒 PARTE 3 — Endurecimento de UFW e iptables (KillerCoda)

### Passo 1 — Estado inicial da UFW
```bash
sudo ufw status
```
**Saída:**
```text
Status: inactive
```
*O firewall foi desativado por omissão — servidor sem qualquer política de filtragem.*

### Passo 2 — Política padrão: Negar entrada padrão
```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### Passo 3 — Permitir SSH (porta 22/tcp)
```bash
sudo ufw allow 22/tcp
```

### Passo 4 — Ativar o UFW
```bash
sudo ufw enable
```
**Saída:**
```text
Firewall is active and enabled on system startup
```

### Passo 5 — Verifique as regras ativas
```bash
sudo ufw status verbose
```
**Saída:**
```text
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), disabled (routed)
New profiles: skip

To                         Action      From
--                         ------      ----
22/tcp                     ALLOW IN    Anywhere
22/tcp (v6)                ALLOW IN    Anywhere (v6)
```

### Passo 6 — Bloquear IP malicioso com iptables
```bash
sudo iptables -A INPUT -s 203.0.113.50 -j DROP
```

| Bandeira | Significado |
| :--- | :--- |
| `-A INPUT` | Adicionados à cadeia INPUT (tráfego de entrada) |
| `-s 203.0.113.50` | Aplicação ao tráfego originado deste IP |
| `-j DROP` | Descarta silenciosamente (sem resposta ao atacante) |

### Passo 7 — Persistência das regras iptables
```bash
sudo mkdir -p /etc/iptables
sudo iptables-save | sudo tee /etc/iptables/rules.v4
```

### Passo 8 — Lista completa de regras
```bash
sudo iptables -L -v
```
**Saída:**
```text
Chain INPUT (policy DROP 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination
28294   14M ufw-before-logging-input  all  --  any    any     anywhere   anywhere
28294   14M ufw-before-input          all  --  any    any     anywhere   anywhere
   59 20269 ufw-after-input           all  --  any    any     anywhere   anywhere
    0     0 DROP       all  --  any    any     203.0.113.50         anywhere

Chain FORWARD (policy DROP 0 packets, 0 bytes)

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
```

---

## 📝 Explicação da Política Aplicada

| Configuração | O que está bloqueado / permitido | Porquê |
| :--- | :--- | :--- |
| `deny incoming` | Todo tráfego externo não mapeado | Princípio *Secure by Default* — superfície de ataque zero |
| `allow outgoing` | Compartimento sem restrições | Permite atualizações e comunicações legítimas do servidor |
| `ALLOW 22/tcp` | SSH permitido de qualquer IP | Gestão remota essencial — única exceção de entrada |
| `DROP 203.0.113.50` | IP malicioso descartado em silêncio | Bloqueia o atacante sem confirmar a existência do host |
| `iptables-save` | Regras persistidas em `/etc/iptables/rules.v4` | Garantir que o bloqueio sobreviva às reinicializações |

---

## 🔗 Referências

* [TryHackMe — Fundamentos de Segurança de Rede](https://tryhackme.com)
* [Playground KillerCoda Ubuntu](https://killercoda.com)
