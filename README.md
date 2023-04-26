# speedtest_ookla

Requisitos de hardware:
     
     8GB de Ram
     50GB de HD
     Placa de rede 1GB
     
 Requisitos para cadastro:
     
     Dominio
     Site
     Email Cooportativo
     IPv4 & IPv6
      

Pacotes essenciais

     su -
     apt install vim wget unzip net-tools psmisc
     
Ajuste no kernel *vim /etc/sysctl.conf* adicione ao final

     # Kernel deve tentar manter o máximo possível de dados em memória principal 
     vm.swappiness = 5
 
     # Evitar que o sistema fique sobrecarregado com muitos dados sujos na memória.
     vm.dirty_ratio = 10
     vm.dirty_background_ratio = 5
 
     # Aumentar o número máximo de conexões simultâneas
     net.core.somaxconn = 65535
 
     # Aumentar o tamanho máximo do buffer de recepção e transmissão de rede
     net.ipv4.tcp_mem = 4096 87380 16777216
     net.core.rmem_max = 16777216
     net.core.wmem_max = 16777216
     net.ipv4.tcp_rmem = 4096 87380 16777216
     net.ipv4.tcp_wmem = 4096 65536 16777216
 
     # Melhorar o desempenho da conexão e a evitar congestionamentos
     net.ipv4.tcp_sack = 1
     net.ipv4.tcp_window_scaling = 1
     net.ipv4.tcp_moderate_rcvbuf = 1
     net.ipv4.tcp_timestamps = 1
 
     # Reduzir o tempo limite de conexão TCP
     net.ipv4.tcp_fin_timeout = 15
 
     # Ativar o escalonamento de fila de recepção de pacotes de rede
     net.core.netdev_max_backlog = 8192
 
     # Aumentar o número máximo de portas locais que podem ser usadas
     net.ipv4.ip_local_port_range = 1024 65535
     
Carregue

     sysctl -p
     
Módulos do Kernel
     
     modprobe -a tcp_illinois
     echo "tcp_illinois" >> /etc/modules
     modprobe -a tcp_westwood
     echo "tcp_westwood" >> /etc/modules
     modprobe -a tcp_htcp
     echo "tcp_htcp" >> /etc/modules

Vamos criar o diretório /usr/local/src/ooklaserver

     mkdir /usr/local/src/ooklaserver  
Vamos baixar o Speedtest dentro do diretório /usr/local/src/ooklaserver

     cd /usr/local/src/ooklaserver
     wget https://install.speedtest.net/ooklaserver/ooklaserver.sh
     chmod +x ooklaserver.sh
     ./ooklaserver.sh install
 Confirme com Y
 
 Agora você já pode testar se o seu servidor está ON através do seguinte link: http://sub.dominio:8080
![image](https://user-images.githubusercontent.com/94009104/234422343-6e0aaaff-7d47-49ae-a680-aaaada1e6dd1.png)

 Pare o serviço
 
     ./ooklaserver.sh stop
 Edite o arquivo de configuração Ookla
 
     vim /usr/local/src/ooklaserver/OoklaServer.properties
 Remova o # e ative o IPv6 (OBRIGATÓRIO) 
 
     OoklaServer.useIPv6 = true
 Descomente a linha e inclua o seu domínio
     
     OoklaServer.allowedDomains = *.ookla.com, *.speedtest.net, *.seudominio.com.br
 Sobreecreva OoklaServer.properties.default com o OoklaServer.properties
 
     cp /usr/local/src/ooklaserver/OoklaServer.properties /usr/local/src/ooklaserver/OoklaServer.properties.default
 Para que o ookla seja tratado como um serviço vamos editar o diretório vim /lib/systemd/system/ooklaserver.service
     
     [Unit]
     Description=OoklaServer-SpeedTest 
     After=network.target

     [Service]
     User=root
     Group=root
     Type=simple
     RemainAfterExit=yes

     WorkingDirectory=/usr/local/src/ooklaserver
     ExecStart=/usr/local/src/ooklaserver/ooklaserver.sh start
     ExecReload=/usr/local/src/ooklaserver/ooklaserver.sh restart
     #ExecStop=/usr/local/src/ooklaserver/ooklaserver.sh stop
     ExecStop=/usr/bin/killall -9 OoklaServer

     TimeoutStartSec=60
     TimeoutStopSec=300

     [Install]
     WantedBy=multi-user.target
     Alias=speedtest.service
 Recarregue o daemon
     
     systemctl daemon-reload
 Vamos ativar o nosso serviço
     
     systemctl enable ooklaserver
     systemctl start ooklaserver
     systemctl status ooklaserver
 Reinicie a máquina

     reboot
 Quando a máquina voltar, verifique se o serviço está UP
 
     systemctl status ooklaserver
 Vamos fazer um teste para ver se o seu servidor acesse o link https://www.ookla.com/pt/host-tester
 ![image](https://user-images.githubusercontent.com/94009104/234431484-fb433fb8-befb-47fb-81ce-a707d03faffe.png)
 Solução para isso é instalarmos o certificado SSL
     
     apt install certbot
     certbot certonly --standalone
 Email (null@gabrielteixeiraconsultoria.com.br ou null@remontti.com.br), Y, N, Seu subdominio e dominio do Speedtest
 
 Edite o arquivo vim /usr/local/src/ooklaserver/OoklaServer.properties para usarmos o certificado gerado
 
 Localize openSSL.server.certificateFile e openSSL.server.privateKeyFile

     openSSL.server.certificateFile = cert.pem
     openSSL.server.privateKeyFile = key.pem
 Subistitua pelo seguinte:
     
     openSSL.server.certificateFile = /etc/letsencrypt/live/SUB.DOMINIO.XXX.XX/fullchain.pem
     openSSL.server.privateKeyFile = /etc/letsencrypt/live/SUB.DOMINIO.XXX.XX/privkey.pem
 Sobreecreva OoklaServer.properties.default com o OoklaServer.properties
 
     cp /usr/local/src/ooklaserver/OoklaServer.properties /usr/local/src/ooklaserver/OoklaServer.properties.default
 Reinicie o ookla (pode demorar)
 
     systemctl  restart ooklaserver.service
 ![image](https://user-images.githubusercontent.com/94009104/234432767-e7e12751-43d7-4d9e-9f97-9a98703a0329.png)
 #Renovar Certificado automaticamente todo dia primeiro
 
 Abra a pasta vim /usr/local/src/ooklaserver/renova-certificado
 
     #!/bin/bash
     # Renova o certificado
     /usr/bin/certbot renew -q
     # Aguarda o certificado renovar
     sleep 30
     # Reinicie o OoklaServer
     /usr/bin/systemctl restart ooklaserver
 De permissão para execução e adicione ao cron, para que ele rode o script toda a meia noite do dia 1º de cada mês
 
     chmod +x /usr/local/src/ooklaserver/renova-certificado
     echo '00 00   1 * *   root    /usr/local/src/ooklaserver/renova-certificado' >> /etc/crontab
 Verifique se a última linha está nosso script
 
     cat /etc/crontab
 Reinicie o cron para ele carregar a nova rotina.

     systemctl restart cron
 



 

