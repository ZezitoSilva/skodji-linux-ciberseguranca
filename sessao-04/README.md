🔐 Sessão 04 — Gestão Segura de Acessos Remotos SSH em Linux

Módulo: Linux e Cibersegurança
Programa: Skodji Digital — Percurso Reskilling
Formador: Péricles Borges
Objetivo de Aprendizagem: OA4 · Aplicar
Data de conclusão: Julho 2026

🎯 Contexto

Nesta sessão protegi o canal de gerenciamento remoto de um servidor Ubuntu, eliminando a autenticação por senha e migrando para autenticação criptográfica com par de chaves Ed25519 — o algoritmo de curvas elípticas mais seguro disponível no OpenSSH.

Plataforma	Objetivo
Playground KillerCoda Ubuntu	Implementação do endurecimento SSH
TryHackMe — Treinamento de Força no Linux	Exercícios de reforço de comandos Linux
⚙️ Passos Executados
Passo 1 — Criar usuário de teste
bash
adduser utilizador_teste

Saída:

Adding user 'utilizador_teste' ...
Creating home directory '/home/utilizador_teste' ...
passwd: password updated successfully

✅ Utilizador criado com home directory em/home/utilizador_teste

Passo 2 — Gerar chaves Ed25519
bash
ssh-keygen -t ed25519

Saída:

Generating public/private ed25519 key pair.
Your identification has been saved in /root/.ssh/id_ed25519
Your public key has been saved in /root/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:xK9mP3qR7nL2vB8wY4tC6hJ0uE5fA1sD root@killercoda
Ficheiro	‐
~/.ssh/id_ed25519	Chave privada — nunca compartilhar
~/.ssh/id_ed25519.pub	Chave pública — instalar no servidor

✅ Par de chaves Ed25519 gerado com sucesso

Passo 3 — Copiar chave pública para o servidor
bash
ssh-copy-id utilizador_teste@localhost

Saída:

Number of key(s) added: 1

Now try logging into the machine with: ssh 'utilizador_teste@localhost'

A chave pública foi instalada em/home/utilizador_teste/.ssh/authorized_keys

✅ Chave pública instalada com sucesso

Passo 4 — Editar o arquivo sshd_config
bash
nano /etc/ssh/sshd_config

Alterações aplicadas:

# Bloquear login directo como root
PermitRootLogin no

# Desativar autenticação por password
PasswordAuthentication no

# Alterar porto padrão
Port 2222
Diretiva	Valentia	Justificação
PermitRootLogin	no	Bloqueia login como root — o atacante nunca sabe o nome de usuário admin
PasswordAuthentication	no	Eliminar força bruta — sem senha não há o que adivinhar
Port	2222	Reduz 99% dos scans automáticos de bots
Passo 5 — Validar sintaxe
bash
sshd -t
# Sem output = configuração válida (nenhum erro encontrado)

✅ Sintaxe válida — seguro para reiniciar o serviço

Passo 6 — Resolver bloqueio de soquete (Ubuntu 22.04+)
bash
systemctl stop ssh.socket
systemctl disable ssh.socket
Removed /etc/systemd/system/sockets.target.wants/ssh.socket.
○ ssh.socket - OpenBSD Secure Shell server socket
     Active: inactive (dead)

No Ubuntu 22.04+, o ssh.socketintercepta o porto 22. É necessário desativá-lo para usar o daemon tradicional com porto personalizado.

Passo 7 — Reiniciar serviço SSH
bash
systemctl restart ssh
systemctl status ssh
● ssh.service - OpenBSD Secure Shell server
     Active: active (running)
     Status: 'Server listening on 0.0.0.0 port 2222.'
     Status: 'Server listening on :: port 2222.'

✅ SSH a escutar no porto 2222 — confirmado

Passo 8 — Testar acesso via chave na porta 2222
bash
ssh -p 2222 utilizador_teste@localhost

Saída:

Welcome to Ubuntu 22.04 LTS (GNU/Linux 5.15.0-76-generic x86_64)

Last login: Mon Jul  1 19:44:01 2026
utilizador_teste@killercoda:~$

✅ Login bem sucedido sem prompt de senha — autenticação por chave Ed25519 funcional

📄 sshd_config — Configuração Final
# /etc/ssh/sshd_config — Configuração Segura
# Skodji Digital — Lab 4 — Zezito Andrade Silva

Port 2222

AddressFamily any
ListenAddress 0.0.0.0
ListenAddress ::

# === SEGURANÇA DE AUTENTICAÇÃO ===

PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
AuthorizedKeysFile .ssh/authorized_keys

KerberosAuthentication no
GSSAPIAuthentication no

Subsystem sftp /usr/lib/openssh/sftp-server
📊 Resumo de Resultados
Passo	Tarefa	Comando	Estado
1	Criar	adduser utilizador_teste	✅ Criado
2	Gerar chaves Ed25519	ssh-keygen -t ed25519	✅ Gerado
3	Copiar chave pública	ssh-copy-id user@localhost	✅ Instalada
4	Editar sshd_config	nano /etc/ssh/sshd_config	✅ 3 s
5	Validar sintaxe	sshd -t	✅ Sem erros
6	Soquete do resolvedor Ubuntu	systemctl stop ssh.socket	✅ Resolvido
7	Reiniciar SSH	systemctl restart ssh	✅ Porto 2222
8	Testar login por chave	ssh -p 2222 user@localhost	✅ Senha Sem
🔗 Referências
TryHackMe — Treinamento de Força no Linux
Playground KillerCoda Ubuntu
Manual do OpenSSH sshd_config
Ed25519 — Comparando chaves SSH
