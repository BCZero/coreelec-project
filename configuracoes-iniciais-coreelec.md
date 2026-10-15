# Relatório de Configuração e Migração do Servidor Home Media

## 1. Especificações do Hardware (TV Box)
* **Marca / Modelo:** BTV Express E13
* **Processador (CPU):** Amlogic S905W2 Quad-Core ARM Cortex-A35 (64 bits)
* **Processador Gráfico (GPU):** ARM Mali-G31 MP2
* **Memória RAM:** 2 GB DDR4
* **Armazenamento Interno (ROM):** 8 GB
* **Sistema Operacional Original:** Android 11
* **Conectividade:** Wi-Fi Dual Band (2.4 GHz e 5.0 GHz), Porta Ethernet (RJ45), USB 2.0, Leitor MicroSD.
* **Resolução:** Suporte a 4K Ultra HD.

## 2. Software e Ferramentas Utilizadas
* **Sistema Operacional Instalado:** CoreELEC 21.3 Omega (Instalado diretamente no SSD).
* **Imagem do Sistema:** `CoreELEC-Amlogic-ne.aarch64-21.3-Omega-Generic.img`
* **Device Tree (DTB) Utilizado:** `s4_s905w2_2g` (Específico para a arquitetura do BTV E13).
* **Ferramenta de Flash:** Rufus v4.15 (Garantindo a gravação sem corromper partições).
* **Transferência de Dados:** WinSCP via protocolo SFTP.

## 3. Ações Realizadas e Validadas (Fase 1)

1. **Boot e Acesso ao Sistema:**
   * Sistema CoreELEC inicializado com sucesso via SSD.
   * Conexão de rede estabelecida via cabo ethernet/Wi-Fi (IP Fixo: `<IP_FIXO_ESCOLHIDO>`).
   
2. **Habilitação de Protocolos (SSH e Samba):**
   * Acesso SSH ativado e validado com o usuário `root` e senha personalizada.
   * Chaves RSA corrigidas no terminal do notebook.
   * Serviço Samba (`smbd` / `nmbd`) ativado no sistema operacional para compartilhamento em rede local.

3. **Instalação do Docker (Nativo):**
   * Devido à impossibilidade temporária de usar a interface do Kodi (controle remoto pendente), o Docker Engine (v24.0.9) foi baixado diretamente dos repositórios oficiais para a arquitetura `aarch64`.
   * Serviço `dockerd` configurado e inicializado com sucesso via `systemd`.
   * Binário do `docker-compose` baixado e ativado.

4. **Estruturação de Diretórios (SSD principal - `/storage`):**
   * Nova árvore de diretórios criada para manter os bancos de dados limpos e organizados:
     * `../media/qbittorrent/downloads`
     * `../media/qbittorrent/animes`
     * `../media/qbittorrent/config`
     * `../media/jellyfin`

5. **Transferência de Mídia:**
   * Utilizado o software **WinSCP** (Plano B seguro contra instabilidades do Windows Explorer) para conectar via porta 22 (SFTP).
   * Migração de aproximadamente 100GB de mídias (filmes, séries e legendas) iniciada com sucesso direto para a pasta `../media`.

## 4. Próximos Passos (Fase 2)

Com a infraestrutura de rede, armazenamento e containerização 100% pronta, o projeto avança para as seguintes etapas:

* [ ] **Testes Individuais de Contêineres:** Subir os serviços isoladamente usando `docker run` para garantir a compatibilidade com a arquitetura Amlogic.
  * *qBittorrent:* Testar prioritariamente a versão `4.3.9` (libtorrent v1) para evitar vazamento de memória na RAM de 2GB.
  * *Jellyfin:* Mapear o banco de dados limpo.
  * *Tailscale:* Configurar túnel de rede local.
* [ ] **Consolidação do Backup:** Após os testes individuais com sucesso, registrar as versões funcionais e gerar o arquivo `docker-compose.yml` definitivo para futuros backups automáticos.
* [ ] **Controle Remoto (Opcional/Futuro):** Configurar o mouse USB temporário para passar do Wizard inicial do Kodi e, posteriormente, mapear via linha de comando o controle remoto infravermelho original (usando o `ir-keytable` e o arquivo `config.ini`).