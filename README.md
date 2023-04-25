# speedtest_ookla

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
# Confirme com Y
# Agora você já pode testar se o seu servidor está ON através do seguinte link: http://sub.dominio:8080
![image](https://user-images.githubusercontent.com/94009104/234422343-6e0aaaff-7d47-49ae-a680-aaaada1e6dd1.png)

     
