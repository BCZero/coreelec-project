# Resumo do Projeto: Servidor de Mídia CoreELEC (Docker, qBittorrent, Jellyfin)

Este documento sintetiza a nossa jornada de configuração, os principais marcos de sucesso, diretórios utilizados e comandos validados para o gerenciamento do seu servidor de mídia auto-hospedado em uma TV Box rodando CoreELEC.

---

## 1. Visão Geral da Arquitetura e Diretórios Principais

O ambiente está estruturado em cima de um SSD externo conectado à TV Box e mapeado nos containers Docker.

* **Caminho Base no SSD:** `/var/media/SSD_DATA`
* **Diretórios Principais Validados:**
  * **qBittorrent (Downloads):** `/var/media/SSD_DATA/qbittorrent/downloads/`
  * **qBittorrent (Animes/Organizados):** `/var/media/SSD_DATA/qbittorrent/animes/`
  * **Configurações gerais:** `/var/media/SSD_DATA/qbittorrent/config/`, `/var/media/SSD_DATA/jellyfin/`, etc.
  * **Mapeamento no Jellyfin:** A raiz `/var/media/SSD_DATA` está mapeada internamente como `/media`, permitindo o acesso direto aos caminhos como `/media/qbittorrent/animes`.

---

## 2. Principais Pontos de Avanço e Sucesso

1. **Estrutura de Containers Estável:** qBittorrent e Jellyfin operando de forma contínua e integrada.
2. **Biblioteca de Animes Configurada:** Criação e indexação bem-sucedida de uma biblioteca dedicada aos animes no Jellyfin apontando para a pasta correta.
3. **Resolução de Legendas Nativa:** 
   * Superação de falhas de compatibilidade do *Bazarr* com o ecossistema `s6-overlay` do CoreELEC.
   * Instalação e configuração bem-sucedida do plugin oficial **OpenSubtitles** diretamente no painel do Jellyfin (utilizando a nova API do `.com`).
   * Validação de busca, download e aplicação de legendas em `.srt` direto pela interface do player/gerenciamento.
4. **Gerenciamento de Arquivos via Terminal:** Domínio dos comandos de navegação, movimentação e tratamento de espaços e caracteres especiais em nomes de pastas/arquivos via SSH.

---

## 3. Comandos Principais Testados e Validados

### Gerenciamento de Containers Docker
* Verificar o status dos containers em execução:
  ```bash
  docker ps
  ```
* Reiniciar o container do Jellyfin para carregar plugins:
  ```bash
  docker restart jellyfin
  ```

### Navegação e Manipulação de Arquivos (SSH)
* Listar arquivos e pastas com identificadores de tipo (diretórios terminam com `/`):
  ```bash
  ls -F /var/media/SSD_DATA/
  ```
* Listar o conteúdo da pasta de downloads do qBittorrent:
  ```bash
  ls -F /var/media/SSD_DATA/qbittorrent/downloads/
  ```
* Mover arquivos ou pastas (com espaços no nome, utilizando **aspas duplas**):
  ```bash
  mv "/var/media/SSD_DATA/qbittorrent/downloads/NOME_DA_PASTA" /var/media/SSD_DATA/qbittorrent/animes/
  ```

### Utilitários do Terminal
* Limpar a tela do terminal do CoreELEC:
  ```bash
  clear
  ```
  *(Ou atalho `Ctrl + L`)*

---

## 4. Próximos Passos Sugeridos
* Automatizar ou padronizar a rotina de mover arquivos concluídos do qBittorrent para as respectivas pastas de mídia.
* Consolidar o hábito de buscar e aplicar legendas diretamente pelo Jellyfin sempre que um novo arquivo for adicionado.