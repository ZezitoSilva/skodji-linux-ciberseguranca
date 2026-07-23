# 🔍 Sessão 02 — Auditoria de Sistemas Linux e Análise Avançada de Logs

**Módulo:** Linux e Cibersegurança  
**Programa:** Skodji Digital — Percurso Reskilling  
**Formador:** Péricles Borges  
**Objetivo de Aprendizagem:** OA2 · Avaliar  
**Data de conclusão:** Julho 2026  

---

## 🎯 Contexto

Nesta sessão atuei como **analista forense** numa investigação de um servidor Apache da **ACME Web Design** que foi alvo de conexões anómalas. O objetivo foi auditar os logs do servidor web para:

- Identificar as ferramentas automatizadas utilizadas pelo atacante
- Rastrear os métodos de ataque e vetores de intrusão
- Descobrir vulnerabilidades exploradas e o utilizador comprometido

---

## 🌐 Ambiente Virtual Utilizado

| Plataforma | URL | Propósito |
|---|---|---|
| TryHackMe — Intro to Logs | https://tryhackme.com/room/introtologs | Compreender a mecânica dos registos do sistema |
| TryHackMe — Linux Server Forensics | https://tryhackme.com/room/linuxserverforensics | Análise forense do servidor comprometido |

---

## ⚙️ Tarefas Executadas

### Parte 1 — Acesso SSH ao Servidor Comprometido

**Objetivo:** Estabelecer ligação remota segura ao servidor alvo para iniciar a auditoria forense.

**Comando executado:**
```bash
ssh fred@10.128.158.215
# Password: FredRules!
```

**Resultado:** O prompt alterou de `root@ip-10-128-175-186` para `fred@ip-10-128-158-215:~$`, confirmando o login com sucesso.

---

### Parte 2 — Análise dos Logs do Servidor Apache

Os logs do Apache estão em `/var/log/apache2/`. As evidências históricas foram encontradas em `access.log.1`.

---

#### Questão A — Quantas ferramentas diferentes fizeram requisições ao servidor?

**Resposta: `2` (DirBuster e Nmap)**

**Comando executado:**
```bash
sudo egrep -oi "Nmap|Nikto|sqlmap|dirbuster|Hydra" access.log.1 | sort -u
```

**Análise:** O filtro retornou a presença de duas ferramentas de scanning:
- **DirBuster** — mapeamento de diretórios ocultos
- **Nmap** — reconhecimento de rede e portas abertas

---

#### Questão B — Qual o caminho solicitado pelo Nmap?

**Resposta: `/nmaplowercheck1618912425`**

**Comando executado:**
```bash
sudo grep -i "nmap" access.log.1 | awk '{print $7}' | sort -u
```

**Análise:** O `awk` isolou o 7º campo (caminho da requisição) das linhas com User-Agent do Nmap. O identificador dinâmico gerado pelo NSE (Nmap Scripting Engine) foi `/nmaplowercheck1618912425`.

---

### Parte 3 — Vetores de Ataque e Identificação do Invasor

#### Questão C — Qual página permite fazer upload de ficheiros?

**Resposta: `contact.php`**

**Comando executado:**
```bash
ls -la /var/www/html
```

**Análise:** A listagem revelou o script `contact.php` configurado incorretamente, permitindo submissões arbitrárias de ficheiros pelo atacante.

---

#### Questão D — Qual o endereço IP que enviou ficheiros para o servidor?

**Resposta: `192.168.56.24`**

**Comando executado:**
```bash
sudo grep -a "POST" access.log.1
```

**Análise:** A flag `-a` forçou o processamento do log como texto normal. O resultado isolou o IP `192.168.56.24` como origem de múltiplos pedidos HTTP `POST` direcionados a `contact.php`.

---

#### Questão E — Quem deixou um aviso de segurança exposto no servidor?

**Resposta: `Fred`**

**Comando executado:**
```bash
cat /var/www/html/resources/development/2021/docs/memo
```

**Análise:** O memorando interno revelou que o administrador **Fred** assinou e expôs um aviso crítico de segurança num diretório público do servidor web.

---

## 📊 Resumo dos Resultados — Linha Temporal do Incidente

| # | Descoberta | Detalhe |
|---|---|---|
| 1 | Ferramentas identificadas | DirBuster + Nmap (2 ferramentas) |
| 2 | Caminho Nmap | `/nmaplowercheck1618912425` |
| 3 | Página vulnerável | `contact.php` (upload sem restrições) |
| 4 | IP do atacante | `192.168.56.24` |
| 5 | Utilizador comprometido | `Fred` (aviso exposto em diretório público) |

---

## 💡 Comandos-Chave desta Sessão

| Comando | Propósito |
|---|---|
| `ssh user@IP` | Ligação remota segura a servidor Linux |
| `egrep -oi "pattern" file` | Busca case-insensitive com múltiplos padrões |
| `grep -i "nmap" file \| awk '{print $7}'` | Isolar campo específico de log |
| `grep -a "POST" file` | Forçar leitura de log binário como texto |
| `ls -la /var/www/html` | Listar ficheiros do servidor web |
| `cat /path/to/file` | Ler conteúdo de ficheiro de texto |
| `sort -u` | Ordenar e remover duplicados |

---

## 🔗 Referências

- [TryHackMe — Intro to Logs](https://tryhackme.com/room/introtologs)
- [TryHackMe — Linux Server Forensics](https://tryhackme.com/room/linuxserverforensics)
- [Apache Log Format Documentation](https://httpd.apache.org/docs/current/logs.html)

---

*Portfólio desenvolvido no âmbito do programa **Skodji Digital** — Módulo Linux e Cibersegurança · Zezito Andrade Silva · 2026*
