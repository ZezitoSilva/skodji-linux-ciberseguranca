# 🔐 Sessão 04 — Gestão Segura de Acessos Remotos SSH em Linux

**Módulo:** Linux e Cibersegurança  
**Programa:** Skodji Digital — Percurso Reskilling  
**Formador:** Péricles Borges  
**Objetivo de Aprendizagem:** OA4 · Aplicar  
**Data de conclusão:** Julho 2026  

---

## 🎯 Contexto

Nesta sessão protegi o canal de gestão remota de um servidor Ubuntu, eliminando a autenticação por password e migrando para autenticação criptográfica com par de chaves **Ed25519** — o algoritmo de curvas elípticas mais seguro disponível no OpenSSH.

| Plataforma | Objectivo |
|---|---|
| KillerCoda Ubuntu Playground | Implementação do hardening SSH |
| TryHackMe — Linux Strength Training | Exercícios de reforço de comandos Linux |

---

## ⚙️ Passos Executados

### Passo 1 — Criar utilizador de teste

```bash
adduser utilizador_teste
```
<img width="589" height="24" alt="image" src="https://github.com/user-attachments/assets/65240309-338a-4638-a62d-9175b4f96b13" />

**Output:**
```
Adding user 'utilizador_teste' ...
Creating home directory '/home/utilizador_teste' ...
passwd: password updated successfully
```
<img width="589" height="299" alt="image" src="https://github.com/user-attachments/assets/fadfcee7-d76e-4993-8b60-23253735df81" />


✅ Utilizador criado com home directory em `/home/utilizador_teste`

---

### Passo 2 — Gerar par de chaves Ed25519

```bash
ssh-keygen -t ed25519
```
<img width="589" height="22" alt="image" src="https://github.com/user-attachments/assets/365be53a-1f81-408e-8936-f59baf96bc70" />


**Output:**
```
Generating public/private ed25519 key pair.
Your identification has been saved in /root/.ssh/id_ed25519
Your public key has been saved in /root/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:xK9mP3qR7nL2vB8wY4tC6hJ0uE5fA1sD root@killercoda
```
<img width="589" height="305" alt="image" src="https://github.com/user-attachments/assets/e87b1cc5-b270-4644-9c73-c6612a11d40e" />


| Ficheiro | Função |
|---|---|
| `~/.ssh/id_ed25519` | Chave **privada** — nunca partilhar |
| `~/.ssh/id_ed25519.pub` | Chave **pública** — instalar no servidor |

<img width="589" height="109" alt="image" src="https://github.com/user-attachments/assets/b0fe48b6-2abc-47de-84a4-89377becce80" />

✅ Par de chaves Ed25519 gerado com sucesso

---

### Passo 3 — Copiar chave pública para o servidor

```bash
ssh-copy-id utilizador_teste@localhost
```

<img width="589" height="18" alt="image" src="https://github.com/user-attachments/assets/9273a7bf-773a-4d8f-bd1f-00ce693f2e2f" />

**Output:**
```
Number of key(s) added: 1

Now try logging into the machine with: ssh 'utilizador_teste@localhost'
```

> A chave pública foi instalada em `/home/utilizador_teste/.ssh/authorized_keys`

<img width="589" height="109" alt="image" src="https://github.com/user-attachments/assets/bf938daa-a44c-4659-9d7c-db7609b4fa29" />

✅ Chave pública instalada com sucesso

---

### Passo 4 — Editar o ficheiro sshd_config

```bash
nano /etc/ssh/sshd_config
```
<img width="589" height="15" alt="image" src="https://github.com/user-attachments/assets/0b979f8d-5266-45b2-87a9-5b414e80590f" />


**Alterações aplicadas:**

```
# Bloquear login directo como root
PermitRootLogin no

# Desativar autenticação por password
PasswordAuthentication no

# Alterar porto padrão
Port 2222
```

| Diretiva | Valor | Justificação |
|---|---|---|
| `PermitRootLogin` | `no` | Bloqueia login como root — atacante nunca sabe o username admin |
| `PasswordAuthentication` | `no` | Elimina força bruta — sem password não há o que adivinhar |
| `Port` | `2222` | Reduz 99% dos scans automáticos de bots |

<img width="587" height="88" alt="image" src="https://github.com/user-attachments/assets/dfd43d0f-cf41-49a8-9d05-1de29cf98d8d" />

---

### Passo 5 — Validar sintaxe

```bash
sshd -t
```
<img width="589" height="14" alt="image" src="https://github.com/user-attachments/assets/ad4cfa5f-54a8-4755-9f42-9e4fef56b538" />

```
# Sem output = configuração válida (nenhum erro encontrado)
```

✅ Sintaxe válida — seguro para reiniciar o serviço

---

### Passo 6 — Resolver bloqueio de socket (Ubuntu 22.04+)

```bash
systemctl stop ssh.socket
systemctl disable ssh.socket
```
<img width="589" height="17" alt="image" src="https://github.com/user-attachments/assets/8e5ccf47-e192-4a59-abfd-a3f125c52b27" />

```
Removed /etc/systemd/system/sockets.target.wants/ssh.socket.
○ ssh.socket - OpenBSD Secure Shell server socket
     Active: inactive (dead)
```

> Em Ubuntu 22.04+, o `ssh.socket` intercepta o porto 22. É necessário desactivá-lo para usar o daemon tradicional com porto personalizado.

---

### Passo 7 — Reiniciar serviço SSH

```bash
systemctl restart ssh
systemctl status ssh
```
<img width="589" height="16" alt="image" src="https://github.com/user-attachments/assets/5997edf1-c280-4531-ab12-df57168243b3" />

```
● ssh.service - OpenBSD Secure Shell server
     Active: active (running)
     Status: 'Server listening on 0.0.0.0 port 2222.'
     Status: 'Server listening on :: port 2222.'
```

<img width="589" height="37" alt="image" src="https://github.com/user-attachments/assets/020f2d67-79db-4af7-9e56-bd25bdf7b905" />

✅ SSH a escutar no porto 2222 — confirmado

---

### Passo 8 — Testar acesso via chave na porta 2222

```bash
ssh -p 2222 utilizador_teste@localhost
```

**Output:**
```
Welcome to Ubuntu 22.04 LTS (GNU/Linux 5.15.0-76-generic x86_64)

Last login: Thu Jul  23 09:42:16 2026
utilizador_teste@killercoda:~$
```

<img width="589" height="50" alt="image" src="https://github.com/user-attachments/assets/0629956e-4e37-4702-8836-3e479e91b108" />

✅ **Login bem-sucedido sem prompt de password — autenticação por chave Ed25519 funcional**

---

## 📊 Resumo de Resultados

| Passo | Tarefa | Comando | Estado |
|---|---|---|---|
| 1 | Criar utilizador | `adduser utilizador_teste` | ✅ Criado |
| 2 | Gerar chaves Ed25519 | `ssh-keygen -t ed25519` | ✅ Gerado |
| 3 | Copiar chave pública | `ssh-copy-id user@localhost` | ✅ Instalada |
| 4 | Editar sshd_config | `nano /etc/ssh/sshd_config` | ✅ 3 diretivas |
| 5 | Validar sintaxe | `sshd -t` | ✅ Sem erros |
| 6 | Resolver socket Ubuntu | `systemctl stop ssh.socket` | ✅ Resolvido |
| 7 | Reiniciar SSH | `systemctl restart ssh` | ✅ Porto 2222 |
| 8 | Testar login por chave | `ssh -p 2222 user@localhost` | ✅ Sem password |

---

## 🔗 Referências

- [TryHackMe — Linux Strength Training](https://tryhackme.com/room/linuxstrengthtraining)
- [KillerCoda Ubuntu Playground](https://killercoda.com/playgrounds/scenario/ubuntu)
- [OpenSSH sshd_config Manual](https://man.openbsd.org/sshd_config)
- [Ed25519 — Comparing SSH Keys](https://goteleport.com/blog/comparing-ssh-keys/)

---

*Portfólio desenvolvido no âmbito do programa **Skodji Digital** — Módulo Linux e Cibersegurança · Zezito Andrade Silva · 2026*
