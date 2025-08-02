# My Awesome Linux
![GitHub last commit (branch)](https://img.shields.io/github/last-commit/bsshenrique/my-awesome-linux/main)

**My Awesome Linux** é um repositório pessoal descrevendo minha visão sobre o uso do Linux como desktop.

> [!TIP]
> Cada distribuição Linux tem a sua própria filosofia e é focada em um propósito.  
> Teste diferentes distribuições até encontrar a que você julgar ideal.

Por exemplo, considero:
- [Alpine Linux](https://www.alpinelinux.org) - Containers e testes;
- [Arch Linux](https://archlinux.org) - Desktop;
- [Fedora](https://fedoraproject.org) - Desktop e *workstation*;
- [Tails](https://tails.net) - Privacidade;
- [Whonix](https://www.whonix.org) - Privacidade.


O Linux é amplamente sustentado por diversas comunidades.  
Se você é um entusiasta, veja alguns sites úteis:
- Distribuições
  - [DistroSea](https://distrosea.com)
  - [DistroWatch](https://distrowatch.com)
  - [The Linux Kernel Archives](https://www.kernel.org)
  - [Wikipedia - List of Linux distributions](https://en.wikipedia.org/wiki/List_of_Linux_distributions)
- Documentações
  - [Arch manual pages](https://man.archlinux.org)
  - [ArchWiki](https://wiki.archlinux.org)
  - [Linux man pages online](https://man7.org/linux/man-pages/index.html)
- Listas
  - [ArchWiki - List of applications](https://wiki.archlinux.org/title/List_of_applications)
  - [Awesome Linux Software](https://github.com/luong-komorebi/Awesome-Linux-Software)
  - [Linux Guide](https://github.com/mikeroyal/Linux-Guide)
  - [LinuxLinks](https://www.linuxlinks.com)
- Notícias
  - [9to5Linux](https://9to5linux.com)
  - [Diolinux](https://diolinux.com.br)
  - [GamingOnLinux](https://www.gamingonlinux.com)
  - [It's FOSS](https://itsfoss.com)


## Tópicos
- [Arch Linux](#arch-linux)
  - [Instalação](#instalação)
  - [Pós-instalação](#pós-instalação)
- [Logs](#logs)
- [pacman](#pacman)
- [Sistemas de Arquivos](#sistemas-de-arquivos)
- [Virtualização](#virtualização)


## Arch Linux
### Instalação
> [!IMPORTANT]
> Evite tutoriais, seu dispositivo e uso certamente são diferentes de outros.  
> Utilize a [documentação de instalação](https://wiki.archlinux.org/title/installation_guide) ou o [archinstall](https://wiki.archlinux.org/title/Archinstall) para uma instalação guiada.  
> O artigo abaixo complementa, mas não substitui a documentação de instalação.

#### Particionamento do dispositivo
Usando o [GPT fdisk](https://wiki.archlinux.org/title/GPT_fdisk), o layout abaixo será construído.

| Partição | Setor              | Tipo | Dispositivo  | Nome   | Montagem  | Formato |
| :------: | :----------------: | :--: | :----------: | :----: | :-------: | :-----: |
| 1        | default à +1GiB    | EF00 | /dev/device1 | EFI    | /mnt/boot | FAT32   |
| 2        | default à default  | 8304 | /dev/device2 | system | /mnt      | BTRFS   |

```bash
# Usando o gdisk
gdisk /dev/device

# Usando o sgdisk
sgdisk --zap-all \
  --new=1:0:+1GiB --typecode=1:EF00 --change-name=1:EFI \
  --new=2:0:0 --typecode=2:8304 --change-name=2:system \
  --print \
  /dev/device
```

> [!NOTE]
> O layout utilizado para os subvolumes é o mesmo usado pelo `archinstall` e sugerido pelo [Snapper](https://wiki.archlinux.org/title/Snapper#Suggested_filesystem_layout).  
> Esse layout garante um isolamento eficiente dos subvolumes.

```bash
mkfs.fat -F 32 -n EFI -v /dev/device1
mkfs.btrfs --label system -v /dev/device2

mount LABEL=system -t btrfs /mnt

btrfs subvolume create /mnt/{@,@home,@log,@pkg,@.snapshots}

umount -R /mnt

MOUNT_OPTIONS="X-mount.mkdir,defaults,compress=zstd:1,noatime"

mount -o $MOUNT_OPTIONS,subvol=@ -t btrfs LABEL=system /mnt
mount -o $MOUNT_OPTIONS,subvol=@home -t btrfs LABEL=system /mnt/home
mount -o $MOUNT_OPTIONS,subvol=@log -t btrfs LABEL=system /mnt/var/log
mount -o $MOUNT_OPTIONS,subvol=@pkg -t btrfs LABEL=system /mnt/var/cache/pacman/pkg
mount -o $MOUNT_OPTIONS,subvol=@.snapshots -t btrfs LABEL=system /mnt/.snapshots
mount -m -t vfat -o dmask=0027,fmask=0137 /dev/device1 /mnt/boot
```

#### Pacotes essenciais
Além dos pacotes essenciais, instale o [microcode](https://wiki.archlinux.org/title/Microcode) correspondente.

`pacstrap -K /mnt amd-ucode base linux linux-firmware nano networkmanager`

#### Localização
Edite o `/etc/locale.gen`:

```plaintext
# Habilite as opções
en_US.UTF-8 UTF-8
pt_BR.UTF-8 UTF-8
```

```bash
locale-gen
echo "LANG=en_US.UTF-8" >> /etc/locale.conf
echo "KEYMAP=br-abnt2" >> /etc/vconsole.conf
```

> [!TIP]
> Altere as configurações de formato utilizando o ambiente gráfico e após a instalação do sistema.

#### Boot loader
Instale o `systemd-boot` utilizando o comando `bootctl install`.

Configure o `esp/loader/loader.conf` conforme o exemplo [loader configuration](https://wiki.archlinux.org/title/Systemd-boot#Loader_configuration).  

```
default       arch.conf
timeout       4
console-mode  max
editor        no
```

Adicione a *entry* `esp/loader/entries/arch.conf` conforme o exemplo [adding loaders](https://wiki.archlinux.org/title/Systemd-boot#Adding_loaders).

> [!TIP]
> Ao usar algum *microcode*, como a `amd-ucode.img`, deve-se sempre referenciá-lo antes do `initrd` principal.

> [!TIP]
> Para utilizar um subvolume como um ponto de montagem, faça o apontamento no *loader*.  
> Confira o exemplo [mounting subvolume as root](https://wiki.archlinux.org/title/Btrfs#Mounting_subvolume_as_root).

```sh
title   Arch Linux
linux   /vmlinuz-linux
initrd  /amd-ucode.img
initrd  /initramfs-linux.img
options root=UUID=XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX rootflags=subvol=@ rw
```


> [!TIP]
> `UUID`, `PARTUUID` ou semelhante são coisas diferentes.
>
> É possível consultar as identificações de `/dev/root_partition` usando:  
> `ls -l /dev/disk/by-*`  
> `blkid /dev/root_partition`
>
> Também é possível usar diretamente o caminho do dispositivo:  
> `options root=/dev/root_partition rw`

Valide a configuração `bootclt list`.

Realize o `umount -R /mnt` e reinicie o sistema.

### Pós-instalação
#### Configurações básicas
Instale alguns dos pacotes básicos:

```plaintext
base-devel
btop
btrfs-progs
curl
fastfetch
firefox
git
gnome
gnome-tweaks
less
noto-fonts
noto-fonts-emoji
noto-fonts-extra
pipewire
pipewire-alsa
pipewire-audio
pipewire-jack
pipewire-pulse
power-profiles-daemon
smartmontools
sudo
wget
which
wireplumber
xdg-utils
zip
zsh
```

O `root` deve ser usado apenas em tarefas específicas ao `root`.  
Crie um novo usuário e permita que o grupo `wheel` acesse o `sudo`.  
Alguns outros exemplos: [sudo](https://wiki.archlinux.org/title/sudo#Example_entries).

```bash
useradd -m -G wheel -s /bin/bash usuario
passwd usuario

sudo EDITOR=nano visudo
# Remova o comentário
# %wheel      ALL=(ALL:ALL) ALL
```

Defina os *hosts* em `/etc/hosts`:

```plaintext
127.0.0.1       localhost
::1             localhost ip6-localhost ip6-loopback
ff02::1         ip6-allnodes
ff02::2         ip6-allrouters
```

```bash
sudo systemctl enable gdm.service
sudo systemctl enable NetworkManager
reboot
```

#### Drivers
Leia o artigo [AMDGPU](https://wiki.archlinux.org/title/AMDGPU) e entenda o que é necessário.  
Os pacotes necessários provavelmente serão: `lib32-mesa`, `lib32-vulkan-radeon`, `mesa` e `vulkan-radeon`.

Configure também o [lm_sensors](https://wiki.archlinux.org/title/Lm_sensors).

#### Zsh
Instale o [Oh My Zsh](https://github.com/ohmyzsh/ohmyzsh).

> [!TIP]
> Ao usar o `Oh My Zsh` sempre realize a instalação de temas e *plugins* por meio dele.
> 
> É possível criar um script para atualizar os temas e *plugins* instalados.  
> Basta um `git pull` nos diretórios em `~/.oh-my-zsh/custom/plugins/` e `~/.oh-my-zsh/custom/themes/`.

Instale o tema [Powerlevel10k](https://github.com/romkatv/powerlevel10k).  
Instale os *plugins* [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions) e [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting).

> [!TIP]
> O comportamento padrão de sugestão após colar algo é bastante irritante e ele pode ser removido conforme o exemplo abaixo.

```bash
plugins=(...)

source $ZSH/oh-my-zsh.sh

# User configuration
ZSH_AUTOSUGGEST_CLEAR_WIDGETS+=(bracketed-paste)
```
#### GNOME
Instale o [Extension Manager](https://flathub.org/apps/com.mattjakeman.ExtensionManager) pelo GNOME Software (Flathub).  
Instale e/ou ative as extensões:  
[Cliboard Indicator](https://extensions.gnome.org/extension/779/clipboard-indicator)  
[Removable Drive Menu](https://extensions.gnome.org/extension/7/removable-drive-menu)  
[System Monitor](https://extensions.gnome.org/extension/6807/system-monitor)

#### nano
Ative a exibição do número de linhas no editor: `echo "set linenumbers" > ~/.nanorc`.

#### nftables
`nftables` é um subsistema do *kernel* Linux desenvolvido para substituir e unificar as implementações `{ip,ip6,arp,eb}tables`.  
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
nft list ruleset                      # Lista todas as regras

nft flush ruleset                     # Remove todas as regras

nft -c -f arquivo.conf                # Valida um arquivo de configuração
nft -f arquivo.conf                   # Carrega um conjunto de regras
# -c, --check
# -f, --file

nft list tables
nft list table familia nome_tabela    # Lista todas as regras de uma tabela

nft add table familia nome_tabela

nft delete table familia nome_tabela

nft flush table familia nome_tabela   # Remove todas as regras de uma tabela
```

Faça a instalação do `iptables-nft` conforme instruções no artigo [nftables](https://wiki.archlinux.org/title/Nftables).  
Edite a configuração `/etc/nftables.conf` conforme sua necessidade.


## Logs
O diretório `/var/log` tem como propósito armazenar *logs* do sistema e de serviços.  
De maneira geral, os *logs* no Linux são acessados por:

```bash
dmesg                         # dmesg, display message, buffer de logs do kernel
dmesg | grep -i "error"

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


## Sistemas de arquivos
Consulte o [artigo](./articles/file-systems.md).


## Virtualização
Máquinas virtuais e containers proporcionam ambientes controlados, de rápida implementação e descartáveis.

### Container
Para utilizar containers, uma maneira bem simples é por meio do Docker:  
`sudo pacman -S docker`  
`sudo systemctl enable docker.service`  
`sudo systemctl start docker.service`  
`sudo usermod -aG docker $USER`

Encerre a sessão e em seguida entre novamente.

### Máquinas virtuais
Consulte o [artigo](./articles/virtual-machines.md).


