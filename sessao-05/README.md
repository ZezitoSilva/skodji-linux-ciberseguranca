## 🔍 Sessão 05 — Análise de Vulnerabilidades em Linux e Ferramentas de Auditoria

**Módulo:** Linux e Cibersegurança
**Programa:** Skodji Digital — Percurso Reskilling
**Formador:** Péricles Borges
**Objetivo de Aprendizagem:** OA5 · Criar
**Data de conclusão:** Julho 2026

---

### 🛡️ Contexto

Nesta sessão atuei como auditor de sistemas, executando um exame de auditoria técnica automatizada com o objetivo de identificar desvios de conformidade de um servidor Linux em relação aos standards de segurança recomendados (CIS Benchmarks).

- Instalar e configurar a ferramenta de auditoria Lynis
- Executar uma auditoria completa ao sistema operativo
- Registar o Hardening Score inicial e os avisos/sugestões encontrados
- Investigar e documentar a correção de duas vulnerabilidades críticas

---

### 🌐 Ambiente Virtual Utilizado

| Plataforma | URL | Propósito |
|---|---|---|
| TryHackMe — Linux Process Analysis | [tryhackme.com/room/linuxprocessanalysis](https://tryhackme.com/room/linuxprocessanalysis) | Contexto de análise de processos e auditoria em Linux |
| KillerCoda Ubuntu Playground | [killercoda.com/playgrounds/scenario/ubuntu](https://killercoda.com/playgrounds/scenario/ubuntu) | Terminal Linux gratuito no browser, usado para correr o Lynis |

---

### 🗂️ Tarefas Executadas

**Parte 1 — Instalação da Ferramenta de Auditoria (Lynis)**

Objetivo: preparar o ambiente instalando o Lynis, a ferramenta open-source de auditoria de segurança e compliance da CISOfy.

Comando executado:
```bash
sudo apt update && sudo apt install lynis -y
```

Resultado: pacote `lynis` (versão 3.0.9) instalado com sucesso, sem interação manual necessária.

---

**Parte 2 — Execução da Auditoria Completa do Sistema**

Objetivo: correr a auditoria completa ao sistema operativo e acompanhar as várias fases do exame (autenticação, kernel, sistema de ficheiros, rede, SSH, logging).

Comando executado:
```bash
sudo lynis audit system
```

Resultado: 262 testes executados, cobrindo todas as categorias de segurança do sistema, com output detalhado gravado em `/var/log/lynis-report.dat` e `/var/log/lynis.log`.

---

**Parte 3 — Registo dos Resultados Finais**

Objetivo: localizar a secção de resultados finais do Lynis e registar o Hardening Score, os Warnings e as Suggestions.

Resultado:

| Métrica | Valor |
|---|---|
| **Hardening Score (índice inicial)** | **60 / 100** |
| **Warnings** | **1** |
| **Suggestions** | **50** |

Warning encontrado:
```
! Found one or more vulnerable packages. [PKGS-7392]
https://cisofy.com/lynis/controls/PKGS-7392/
```

Análise: o sistema apresenta um nível de hardening moderado. O único warning aponta para pacotes desatualizados/vulneráveis, corrigível com uma atualização de rotina do sistema. As 50 suggestions distribuem-se sobretudo pelas áreas de Authentication, SSH, Kernel Hardening e Filesystem.

---

**Parte 4 — Investigação de Vulnerabilidades Críticas Selecionadas**

Objetivo: escolher 2 Suggestions críticas nas áreas de Authentication ou Filesystem e pesquisar a correção recomendada na base de dados Cisofy.

**Vulnerabilidade 1 — `AUTH-9262` (Authentication)**
> *Install a PAM module for password strength testing like pam_cracklib or pam_passwdqc*
> https://cisofy.com/lynis/controls/AUTH-9262/

Análise: sem um módulo PAM de verificação de robustez, o sistema aceita palavras-passe fracas, aumentando a exposição a ataques de força bruta.

Correção recomendada:
```bash
sudo apt install libpam-pwquality -y
# Em /etc/pam.d/common-password, adicionar/ajustar:
# password requisite pam_pwquality.so retry=3 minlen=12 difok=3
```

**Vulnerabilidade 2 — `FILE-6310` (Filesystem)**
> *To decrease the impact of a full /home, /tmp and /var file system, place them on a separate partition*
> https://cisofy.com/lynis/controls/FILE-6310/

Análise: com `/home`, `/tmp` e `/var` no mesmo volume que a raiz, o esgotamento de espaço em qualquer um destes diretórios pode comprometer a estabilidade de todo o sistema.

Correção recomendada:
- Sistemas novos: criar partições/volumes dedicados para `/home`, `/tmp` e `/var` na instalação.
- Sistemas existentes: migrar o conteúdo para novas partições e atualizar o `/etc/fstab`.
- Alternativa para `/tmp`: usar `tmpfs` em memória com limite de tamanho definido.

---

### 📊 Resumo dos Resultados — Linha Temporal da Auditoria

| # | Descoberta | Detalhe |
|---|---|---|
| 1 | Hardening Score inicial | 60 / 100 |
| 2 | Warnings | 1 — pacotes vulneráveis (`PKGS-7392`) |
| 3 | Suggestions | 50 |
| 4 | Vulnerabilidade crítica (Authentication) | `AUTH-9262` — falta de módulo PAM de robustez de password |
| 5 | Vulnerabilidade crítica (Filesystem) | `FILE-6310` — `/home`, `/tmp`, `/var` não segregados |

---

### 🔑 Comandos-Chave desta Sessão

| Comando | Propósito |
|---|---|
| `sudo apt update && sudo apt install lynis -y` | Instalar o Lynis |
| `sudo lynis audit system` | Executar a auditoria completa ao sistema |
| `lynis show details TEST-ID` | Consultar detalhes de um teste específico |
| `less /var/log/lynis.log` | Consultar o log completo da auditoria |
| `cat /var/log/lynis-report.dat` | Consultar o relatório estruturado |

---

### ✅ Conclusão

A auditoria confirmou um sistema com hardening moderado (60/100), sem falhas críticas imediatas, mas com margem clara de melhoria em autenticação e segregação de sistema de ficheiros. As correções prioritárias — instalação de `pam_pwquality` e segregação de partições — foram documentadas com base nos controlos oficiais da Cisofy.

---

### 🔗 Referências

- [TryHackMe — Linux Process Analysis](https://tryhackme.com/room/linuxprocessanalysis)
- [KillerCoda Ubuntu Playground](https://killercoda.com/playgrounds/scenario/ubuntu)
- [Lynis — CISOfy Controls Database](https://cisofy.com/lynis/controls/)
