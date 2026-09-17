# Mini-investigação de Logs de Autenticação SSH

## Objetivo

Analisar eventos de autenticação via SSH em um servidor Ubuntu Linux, identificando tentativas de acesso com usuário inválido, senha incorreta e uma autenticação bem-sucedida.

## Ambiente

* Ubuntu Server em máquina virtual
* Interface de rede em modo Bridge
* Serviço SSH habilitado
* Máquina cliente na mesma rede local

## Cenário

A partir da máquina cliente, foram realizadas diferentes tentativas de autenticação SSH contra o servidor:

1. tentativa utilizando um usuário inexistente;
2. tentativa utilizando um usuário válido com senha incorreta;
3. autenticação bem-sucedida com usuário e senha válidos.

Os eventos gerados foram posteriormente analisados nos logs do servidor.

## Evidências

### Usuário inválido

```text
Sep 17 20:58:55 ubuntuserver sshd-session[2907]: pam_unix(sshd:auth): check pass; user unknown
Sep 17 20:58:55 ubuntuserver sshd-session[2907]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=192.168.1.8
Sep 17 20:58:57 ubuntuserver sshd-session[2907]: Failed password for invalid user asm from 192.168.1.8 port 65427 ssh2
```

O servidor recebeu uma tentativa de autenticação para o usuário `asm`, que não existia no sistema.

Informações identificadas:

* Usuário: `asm`
* IP de origem: `192.168.1.8`
* Porta de origem: `65427`
* Resultado: falha de autenticação para usuário inexistente

### Usuário válido com senha incorreta

```text
Sep 17 20:59:26 ubuntuserver unix_chkpwd[2914]: password check failed for user (olavo)
Sep 17 20:59:26 ubuntuserver sshd-session[2912]: pam_unix(sshd:auth): authentication failure; logname= uid=0 euid=0 tty=ssh ruser= rhost=192.168.1.8
Sep 17 20:59:27 ubuntuserver sshd-session[2912]: Failed password for olavo from 192.168.1.8 port 65430 ssh2
```

Neste caso, o usuário `olavo` existia no servidor, porém a senha fornecida estava incorreta.

Informações identificadas:

* Usuário: `olavo`
* IP de origem: `192.168.1.8`
* Porta de origem: `65430`
* Resultado: falha de autenticação

### Autenticação bem-sucedida

```text
Sep 17 21:52:43 ubuntuserver sshd-session[3661]: Accepted password for olavo from 192.168.1.8 port 61308 ssh2
Sep 17 21:52:43 ubuntuserver sshd-session[3661]: pam_unix(sshd:session): session opened for user olavo(uid=1000) by olavo(uid=0)
```

O evento `Accepted password` confirma que a autenticação foi aceita. Em seguida, o PAM registra a abertura da sessão do usuário.

Informações identificadas:

* Usuário: `olavo`
* IP de origem: `192.168.1.8`
* Porta de origem: `61308`
* Resultado: autenticação bem-sucedida e sessão aberta

## Análise

Foi possível diferenciar três comportamentos distintos nos registros de autenticação:

* tentativa com usuário inexistente;
* tentativa com usuário válido e senha incorreta;
* autenticação válida seguida da abertura de uma sessão.

Os logs forneceram informações como horário, usuário, endereço IP de origem, porta de origem e resultado da tentativa.

Também foi possível observar que as diferentes conexões partiram do mesmo endereço IP, mas utilizaram portas de origem diferentes.

## Aprendizado

O laboratório ajudou a desenvolver familiaridade com logs de autenticação Linux e com a interpretação de eventos gerados pelo serviço SSH.

Também demonstrou como registros aparentemente simples podem ser utilizados para reconstruir uma sequência de tentativas de acesso e servir como evidência durante uma investigação.

## Próximos passos

* gerar múltiplas tentativas de autenticação;
* filtrar eventos por IP, usuário e resultado;
* contabilizar falhas automaticamente;
* automatizar parte da análise utilizando Python;
* criar uma detecção simples para múltiplas falhas de autenticação.
