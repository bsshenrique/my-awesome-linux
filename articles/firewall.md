# Firewall
## nftables
`nftables` é a estrutura moderna do *kernel* Linux utilizado para classificar pacotes.  
Foi desenvolvido para substituir e unificar as implementações `{ip,ip6,arp,eb}tables`.

No conceito básico do `nftables`, existem principalmente *tables* e *chains*.

*Tables* são conjuntos lógicos de *chains*.  
Toda *table* pertence a uma família.

```plaintext
Família   Finalidade        Substitui
ip	      IPv4              iptables
ip6	      IPv6              ip6tables
inet	    IPv4 e IPv6       iptables e ip6tables
arp	      ARP               arptables
bridge	  Bridge Ethernet   ebtables
```

*Chains* são um conjunto de *rules* armazenados dentro de uma *table*.  
Podem ser classificadas em *base chains*, que atuam em pontos específicos do fluxo de rede do kernel (como *input*, *output*, *forward*), ou em *regular chains*, que são usadas para agrupar *rules* e saltar para elas.

As *rules* são processadas de cima para baixo, em ordem sequencial.  

Comandos básicos para administrar o `nftables`.

```bash
sudo nft list ruleset                      # Lista todas as regras

sudo nft flush ruleset                     # Remove todas as regras

sudo nft -c -f arquivo.conf                # Valida um arquivo de configuração
sudo nft -f arquivo.conf                   # Carrega um conjunto de regras
# -c, --check
# -f, --file

sudo nft list tables
sudo nft list table familia nome_tabela    # Lista todas as regras de uma tabela

sudo nft add table familia nome_tabela

sudo nft delete table familia nome_tabela

sudo nft flush table familia nome_tabela   # Remove todas as regras de uma tabela
```

Faça a instalação do `iptables-nft` conforme instruções no artigo [nftables](https://wiki.archlinux.org/title/Nftables).  
Edite a configuração `/etc/nftables.conf` conforme sua necessidade.
