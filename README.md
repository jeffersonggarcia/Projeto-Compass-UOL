# Projeto-Compass-UOL

**DOCUMENTAÇÃO DA INSTALAÇÃO DO SERVIDOR LINUX COM APACHE E NFS NA AWS**

**Autor:** Jefferson Gomes Garcia  
**Versão 1.0 (09/09/2024)**

## INTRODUÇÃO

Esta documentação fornece um guia passo a passo para a instalação de um servidor com o Sistema Operacional Linux, onde serão instalados os serviços Apache e NFS (Network File System) em um Servidor Cloud hospedado na Amazon AWS.

**Configuração do Servidor Utilizado:**
- Servidor Cloud hospedado na Amazon WS
- Amazon Linux 2 AMI (HVM) - Kernel 5.10, SSD Volume Type
- Instância EC2 da Família t3.small (2vCPU, Memória 2GB)
- 16 GB SSD de Uso Geral (gp3)
- Elastic IP
- Portas de Comunicação Liberadas: 22/TCP, 80/TCP, 111/TCP e UDP, 443/TCP e 2049/TCP.

## INSTALAÇÃO DO NFS E DO APACHE NA AWS

1. **Conecte-se à Instância EC2 via SSH** e atualize o sistema:

    - Para Amazon Linux 2 ou RHEL:
      ```bash
      sudo yum update -y
      ```

    - Para Ubuntu ou Debian:
      ```bash
      sudo apt update && sudo apt upgrade -y
      ```

2. **Instale o NFS (Network File System):**

    - Para Amazon Linux 2 ou RHEL:
      ```bash
      sudo yum install nfs-utils -y
      ```

    - Para Ubuntu ou Debian:
      ```bash
      sudo apt install nfs-kernel-server -y
      ```

3. **Configure o NFS:**

    - Crie o diretório a ser compartilhado:
      ```bash
      sudo mkdir -p /home/jefferson/server-nfs
      ```

    - Configure as permissões do diretório:
      ```bash
      sudo chown -R ec2-user:ec2-user /home/jefferson/server-nfs/
      sudo chmod 755 /home/jefferson/server-nfs/
      ```

    - Edite o arquivo de configuração `/etc/exports`:
      ```bash
      sudo nano /etc/exports
      ```

      - Para permitir o acesso a todos os clientes:
        ```
        /home/jefferson/server-nfs/ *(rw,sync,no_subtree_check)
        ```

      - Para permitir o acesso apenas a IPs específicos:
        ```
        /home/jefferson/server-nfs/ 192.168.1.0/24(rw,sync,no_subtree_check)
        ```

    - Aplique as mudanças:
      ```bash
      sudo exportfs -a
      ```

    - Inicie e habilite o serviço NFS:

      - Para Amazon Linux 2 ou RHEL:
        ```bash
        sudo systemctl start nfs-server
        sudo systemctl enable nfs-server
        ```

      - Para Ubuntu ou Debian:
        ```bash
        sudo systemctl start nfs-kernel-server
        sudo systemctl enable nfs-kernel-server
        ```

4. **Verifique o status do NFS:**

    - Para Amazon Linux 2 ou RHEL:
      ```bash
      sudo systemctl status nfs-server
      ```

    - Para Ubuntu ou Debian:
      ```bash
      sudo systemctl status nfs-kernel-server
      ```

5. **Verifique as regras de segurança da AWS para a porta 2049 (NFS).**

6. **Instale o Apache:**

    - Para Amazon Linux 2 ou RHEL:
      ```bash
      sudo yum install httpd -y
      sudo systemctl start httpd
      sudo systemctl enable httpd
      ```

    - Para Ubuntu ou Debian:
      ```bash
      sudo apt install apache2 -y
      sudo systemctl start apache2
      sudo systemctl enable apache2
      ```

7. **Verifique o status do Apache:**

    - Para Amazon Linux 2 ou RHEL:
      ```bash
      sudo systemctl status httpd
      ```

    - Para Ubuntu ou Debian:
      ```bash
      sudo systemctl status apache2
      ```

8. **Verifique as regras de segurança da AWS para as portas 80 (HTTP) e 443 (HTTPS).**

9. **Abra um navegador e digite o IP público da instância EC2 para ver a página padrão do Apache.**

## MONITORAMENTO DOS SERVIÇOS

1. **Crie o script `check_service.sh`:**

    ```bash
    #!/bin/bash

    # Configuração dos serviços e diretórios de saída
    SERVICES=("httpd" "nfs-server")
    NFS_MOUNT="/home/jefferson/server-nfs/logs"
    LOG_FILE="/var/log/service_monitor.log"
    TIMESTAMP=$(date +"%Y-%m-%d %H:%M:%S")

    # Função para logar mensagens
    log_message() {
        local message="$1"
        echo "$TIMESTAMP - $message" >> "$LOG_FILE"
    }

    # Verifica se o diretório de montagem existe; caso contrário, tenta criar
    if [ ! -d "$NFS_MOUNT" ]; then
        log_message "Diretório $NFS_MOUNT não existe. Tentando criar..."
        mkdir -p "$NFS_MOUNT"
        if [ $? -ne 0 ]; then
            log_message "Não foi possível criar o diretório $NFS_MOUNT. Abortando."
            exit 1
        fi
        log_message "Diretório $NFS_MOUNT criado com sucesso."
    fi

    # Monitoramento dos serviços
    for SERVICE_NAME in "${SERVICES[@]}"; do
        if systemctl is-active --quiet "$SERVICE_NAME"; then
            STATUS="ONLINE"
            MESSAGE="O serviço $SERVICE_NAME está ONLINE."
            OUTPUT_FILE="$NFS_MOUNT/${SERVICE_NAME}_online.txt"
        else
            STATUS="OFFLINE"
            MESSAGE="O serviço $SERVICE_NAME está OFFLINE."
            OUTPUT_FILE="$NFS_MOUNT/${SERVICE_NAME}_offline.txt"
        fi

        # Grava o resultado em um arquivo
        echo "$TIMESTAMP - Serviço: $SERVICE_NAME - Status: $STATUS - $MESSAGE" >> "$OUTPUT_FILE"
        log_message "Resultado para o serviço $SERVICE_NAME gravado em $OUTPUT_FILE."
    done
    ```

2. **Torne o script executável e configure o cron:**

    ```bash
    chmod +x check_service.sh
    sudo nano /etc/crontab
    ```

    Adicione a linha:
    ```
    */5 * * * * root /home/jefferson/check_service.sh
    ```

3. **Verifique os arquivos de saída e execute o script manualmente se necessário:**

    ```bash
    ./check_service.sh
    ```

---

Este `README.md` fornece um guia completo para a instalação e configuração dos serviços no servidor Linux, incluindo a criação e execução do script de monitoramento. Foi desenvolvido como parte de uma atividade de um Projeto de Estágio na empresa Compass-UOL, com o objetivo de implementar uma solução de servidor eficiente para a gestão de dados e serviços web. A documentação inclui instruções detalhadas para garantir uma configuração adequada e funcional do ambiente, cobrindo desde a instalação dos serviços até o monitoramento contínuo dos mesmos.
