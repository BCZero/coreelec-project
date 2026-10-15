### Procedimento de instalação do docker no Coreelec
### Salve esse comando, pois só deu certo depois de várias tentativas!
### Data: 15/10/2026 | segunda criação do CoreElec, após falha severa de inicialização do pen drive (possivelmente causada por desligamento repentino)

mkdir -p /storage/bin /storage/.config/system.d /storage/.docker/data

echo "1. Baixando Docker nativo (aarch64)..."
curl -L -o /storage/docker.tgz "https://download.docker.com/linux/static/stable/aarch64/docker-24.0.9.tgz"

echo "2. Extraindo arquivos..."
tar -xzf /storage/docker.tgz -C /storage
mv -f /storage/docker/* /storage/bin/
rm -rf /storage/docker /storage/docker.tgz

echo "3. Criando o servico do sistema..."
cat << 'EOF' > /storage/.config/system.d/dockerd.service
[Unit]
Description=Docker Daemon
After=network-online.target

[Service]
Type=simple
Environment="PATH=/storage/bin:/usr/sbin:/usr/bin:/sbin:/bin"
ExecStart=/storage/bin/dockerd --data-root=/storage/.docker/data
ExecReload=/bin/kill -s HUP $MAINPID
Restart=always
RestartSec=5
LimitNOFILE=1048576

[Install]
WantedBy=multi-user.target
EOF

echo "4. Iniciando o Docker..."
systemctl daemon-reload
systemctl enable --now dockerd.service

echo 'export PATH=/storage/bin:$PATH' >> /storage/.profile
export PATH=/storage/bin:$PATH

echo "5. Testando a tabela de conteineres..."
sleep 4
docker ps