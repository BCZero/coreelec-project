1. Como desligar o servidor com segurança?
Como você agora tem um sistema Linux real rodando bancos de dados e contêineres Docker (além da transferência de arquivos), nunca puxe a TV Box da tomada. Isso corrompe o SSD e quebra os contêineres.

O CoreELEC é inteligente: ao receber o comando de desligamento, ele avisa o Docker, que salva o estado do qBittorrent e do Jellyfin e encerra tudo com segurança.

Como você já tem o terminal SSH aberto no notebook, os comandos seguros são:

Para Desligar: Digite 
    
    poweroff 
    
e aperte Enter. A TV Box vai desligar completamente.

Para Reiniciar: Digite 

    reboot 

e aperte Enter.

2. É possível fazer o controle remoto funcionar?
Sim, totalmente possível!

Ele parou de funcionar porque as TV Boxes (como a BTV) usam códigos infravermelhos (IR) proprietários. O seu pen drive antigo tinha um arquivo de texto específico que traduzia o sinal do seu controle para o CoreELEC. Como o SSD foi formatado, o sistema perdeu esse "dicionário".

Temos um passo a passo para recriar esse arquivo mapeando botão por botão via SSH usando a ferramenta ir-keytable, mas isso exige ler os logs enquanto você aperta os botões.

Como você está no meio da transferência de 100GB, recomendo não mexermos nos processos do controle agora para não interromper nada. Mas você precisa sair da tela de "Boas-vindas" (Wizard) do Kodi que está travada na TV.

Aqui estão as formas de contornar isso agora:

Método mais fácil (Mouse USB): Pegue qualquer mouse ou teclado USB de computador e espete na porta USB da TV Box. O ponteiro do mouse vai aparecer na TV na mesma hora. Você pode simplesmente clicar em "Next" até concluir o assistente de instalação.

Método Ninja (Via SSH): Você pode usar o terminal para enviar comandos como se fosse o controle remoto.

Para apertar "OK/Selecionar": kodi-send --action="Select"

Para ir para a direita: kodi-send --action="Right"

Para ir para baixo: kodi-send --action="Down"

Qual é o plano: Sugiro que você plugue um mouse USB só para passar dessa tela inicial do Kodi e deixá-lo na tela principal. Assim que terminarmos de subir o qBittorrent (e a cópia dos seus 100GB acabar), mergulhamos na configuração definitiva para reviver o controle remoto original.