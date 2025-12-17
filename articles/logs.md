# Logs
O diretório `/var/log` tem como propósito armazenar *logs* do sistema e de serviços.  
De maneira geral, os *logs* no Linux são acessados por:

```bash
sudo dmesg                    # dmesg, display message, buffer de logs do kernel
sudo dmesg | grep -i "error"

journalctl                    # Ferramenta do systemd para visualizar logs
journalctl _UID=id            # Logs por ID de usuário

journalctl --disk-usage
journalctl --vacuum-size=x    # Limpa os logs mais antigos excedentes ao tamanho de "x"
                              # Ex.: 500M, 1G

journalctl --vacuum-time=x    # Limpa os logs mais antigos do que o período de "x"
                              # Ex.: 7d, 3month, 1h, 1y

journalctl -k                 # Equivalente ao dmesg
journalctl -p n               # Logs por prioridade, de 1~7
journalctl -u name.service    # Logs por serviço
```
