# Configuração do Servidor Samba (CoreELEC) para Acesso no Windows via Tailscale

**Data:** Outubro de 2026
**Cenário:** Acesso de um SSD Externo (`/var/media/SSD_DATA`) conectado a uma TV Box com CoreELEC a partir de um computador com Windows 11.
**Desafios Encontrados:** Bloqueio de rede local pelo roteador da operadora (AP Isolation), restrições do Windows para acesso de convidado e dessincronização de senhas entre Linux e Samba.

---

## 1. Configurando o Samba no CoreELEC (via SSH)

Por padrão, o CoreELEC tenta automatizar os compartilhamentos, mas o Windows pode recusar a conexão. Foi necessário criar uma configuração manual forçando o protocolo SMB2 e expondo explicitamente a pasta do SSD.

**Comando para criar/editar o arquivo:**
```bash
nano /storage/.config/samba.conf

Conteúdo do arquivo samba.conf:

Ini, TOML
[global]
    map to guest = Bad User
    guest account = root
    min protocol = SMB2

[SSD_DATA]
    path = /var/media/SSD_DATA
    available = yes
    browsable = yes
    public = yes
    writable = yes
    guest ok = yes
    force user = root
Nota: A linha min protocol = SMB2 foi crucial para que o Windows 11 aceitasse negociar a conexão sem o erro "não localizado".

2. Definindo a Senha do Samba (O Pulo do Gato)
O Linux possui bancos de dados de senhas separados. A senha utilizada para acessar o terminal SSH não vale automaticamente para o serviço Samba. Foi necessário cadastrar o usuário root no banco do Samba para evitar o Erro de Rede (1208).

Comandos executados no SSH:

Bash
# 1. Cadastrar a senha no Samba (Defina sua senha de preferência)
smbpasswd -a root

# 2. Reiniciar o serviço para aplicar a nova configuração e a nova senha
systemctl restart smbd nmbd
3. Acessando pelo Windows
Devido ao bloqueio de tráfego local pelo roteador da operadora, utilizamos o IP da rede virtual criada pelo Tailscale para estabelecer um túnel direto entre o Windows e o CoreELEC.

Passo a passo do acesso:

No Explorador de Arquivos do Windows, o acesso foi feito diretamente para a pasta destino pelo IP do Tailscale:
\\<IP_DO_TAILSCALE>\ssd_data

Na tela de credenciais do Windows, foi preenchido:

Usuário: \root

Senha: <sua_senha_cadastrada>

Por que usar a barra \ antes do usuário?
Ao colocar \root, forçamos o Windows a utilizar um usuário local da máquina de destino. Sem a barra, o Windows tentaria enviar o nome do seu computador (ex: MEU-PC\root), o que causaria "Acesso Negado".

4. (Opcional) Limpeza de Cache de Rede
Durante os testes de erro de autenticação (Erro 0x80070035), foi necessário limpar tentativas travadas usando o Prompt de Comando do Windows:

DOS
net use \\<IP_DO_TAILSCALE> /delete
Também foram removidas credenciais antigas armazenadas no "Gerenciador de Credenciais" do Painel de Controle do Windows.


Parabéns pela persistência! Agora o seu servidor de mídia está perfeitamente acessível, permitindo que você organize seus arquivos, baixe novos filmes e continue a evoluir a sua trilha de DevOps direto do seu notebook. Se quiser partir para a migração do `docker-compose` amanhã ou outro dia, é só me chamar!