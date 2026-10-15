# Resumo do Projeto: Bot Telegram para qBittorrent no CoreELEC

Este documento registra os passos, configurações e o código final utilizados para criar um bot no Telegram capaz de receber links *magnet* e enviá-los diretamente para um servidor qBittorrent rodando em uma TV Box com CoreELEC (via Docker).

## 1. Ambiente e Estrutura
- **Sistema:** TV Box rodando CoreELEC.
- **Armazenamento:** SSD externo mapeado em `/var/media/SSD_DATA`.
- **Servidor Torrent:** qBittorrent acessível via `http://localhost:8080`.
- **Bot:** Escrito em Python usando a biblioteca `python-telegram-bot` e a biblioteca `requests`, rodando em um container Docker próprio.

## 2. Configurações que Deram Certo

### Docker (Rede e Volumes)
Para que o bot conseguisse enxergar o qBittorrent localmente e o armazenamento do SSD, o container do bot foi criado com os seguintes parâmetros fundamentais:
- `--net=host`: Permite que o bot acesse o qBittorrent usando `localhost` (ou `127.0.0.1`).
- `-v /var/media/SSD_DATA:/var/media/SSD_DATA`: Mapeia o SSD externo para que o comando `/status` do bot possa ler o espaço em disco corretamente.

**Comandos de deploy do Docker:**
```bash
docker stop telegram-bot
docker rm telegram-bot
docker build -t coreelec-telegram-bot .
docker run -d --restart always --name telegram-bot --net=host -v /var/media/SSD_DATA:/var/media/SSD_DATA coreelec-telegram-bot
```

### Segurança e Autenticação do qBittorrent
Para contornar erros de permissão (`401 Unauthorized` e `403 Forbidden`), duas frentes foram configuradas:

1. **Na Interface Web do qBittorrent (Opções > Web UI):**
   - Foi marcada a opção: `Ignorar autenticação pra clientes no hospedeiro local`.
   - Foi marcada a opção: `Ignorar autenticação pra clientes nas sub-redes dos IPs da lista branca` adicionando a faixa do Docker e rede local (ex: `172.16.0.0/12, 192.168.0.0/16, 127.0.0.1`).

2. **Na lógica do Script (O pulo do gato):**
   - O login na API precisava manter o *cookie* de sessão ativo. Para isso, utilizamos o `requests.Session()`.
   - A API moderna do qBittorrent não retorna a string `"Ok."` ao fazer login com sucesso, ela retorna um corpo vazio com o **Status HTTP 204 (No Content)** ou **200 (OK)**. A validação do login no script foi ajustada para ler o `status_code` em vez do texto.

## 3. O Script Final (`bot.py`)

Este é o código definitivo que está rodando no seu container e funcionando perfeitamente:

```python
import os
import shutil
import requests
from telegram import Update
from telegram.ext import ApplicationBuilder, ContextTypes, CommandHandler, MessageHandler, filters

# === CONFIGURAÇÕES ===
TOKEN = "SEU_TELEGRAM_BOT_TOKEN"
ADMIN_USER_ID = SEU_TELEGRAM_USER_ID  # Substitua pelo seu ID numérico do Telegram

QBIT_HOST = "http://127.0.0.1:8080"
QBIT_USER = "admin"  # Seu usuário do qBittorrent
QBIT_PASS = "sua_senha_aqui"  # Sua senha real configurada no qBittorrent

async def status_comando(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Retorna o status de armazenamento do SSD Externo."""
    if update.effective_user.id != ADMIN_USER_ID:
        return

    total, used, free = shutil.disk_usage("/var/media/SSD_DATA")
    total_gb = total // (2**30)
    used_gb = used // (2**30)
    free_gb = free // (2**30)
    
    msg = (
        f"STATUS DO SERVIDOR COREELEC\n\n"
        f"SSD Externo (/var/media/SSD_DATA):\n"
        f"- Total: {total_gb} GB\n"
        f"- Usado: {used_gb} GB\n"
        f"- Livre: {free_gb} GB ({(free/total)*100:.1f}%)"
    )
    await update.message.reply_text(msg)

async def receber_mensagem(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Recebe links magnet e envia para a API do qBittorrent."""
    if update.effective_user.id != ADMIN_USER_ID:
        return

    texto = update.message.text.strip()
    
    if texto.startswith("magnet:?"):
        try:
            # Usa Session() para reter o cookie de login entre as requisições
            session = requests.Session()
            
            # 1. Faz o login na API do qBittorrent
            login_url = f"{QBIT_HOST}/api/v2/auth/login"
            login_response = session.post(login_url, data={'username': QBIT_USER, 'password': QBIT_PASS})
            
            # O qBittorrent retorna 200 OK ou 204 No Content quando o login é bem-sucedido
            if login_response.status_code not in [200, 204]:
                await update.message.reply_text("⚠️ Falha de autenticação na API do qBittorrent. Verifique o usuário e a senha no código.")
                return

            # 2. Envia o torrent usando a mesma sessão autenticada (com o cookie válido)
            add_url = f"{QBIT_HOST}/api/v2/torrents/add"
            payload = {
                'urls': texto,
                'savepath': '/downloads' # Ajuste o caminho final do qBittorrent se necessário
            }
            response = session.post(add_url, data=payload)
            
            if response.status_code == 200:
                await update.message.reply_text("📥 Magnet link adicionado com sucesso ao qBittorrent!")
            else:
                await update.message.reply_text(f"⚠️ Erro ao adicionar torrent. Código: {response.status_code}")
        except Exception as e:
            await update.message.reply_text(f"❌ Falha na conexão com o qBittorrent: {e}")
    else:
        await update.message.reply_text("Envie um link magnet válido (começados com magnet:?).")

def main():
    app = ApplicationBuilder().token(TOKEN).build()
    
    # Comandos e manipuladores
    app.add_handler(CommandHandler("status", status_comando))
    app.add_handler(MessageHandler(filters.TEXT & (~filters.COMMAND), receber_mensagem))
    
    print("Bot do Telegram iniciado...")
    app.run_polling()

if __name__ == "__main__":
    main()
```