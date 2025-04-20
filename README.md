# My Awesome Linux
![GitHub last commit (branch)](https://img.shields.io/github/last-commit/bsshenrique/my-awesome-linux/main)

**My Awesome Linux** é um repositório pessoal descrevendo o que considero em um ambiente Desktop Linux.


## Tópicos
- [Sites úteis](#sites-úteis)
- [Distribuições Linux](#distribuições-linux)
- [Arch Linux](#arch-linux)
  - [Instalação](#instalação)
  - [Configurações adicionais](#configurações-adicionais)
- [Logs](#logs)
- [Sistema de arquivos](#sistema-de-arquivos)
- [Virtualização](#virtualização)


## Sites Úteis
[Diolinux](https://diolinux.com.br/)  
[DistroSea](https://distrosea.com/)  
[DistroWatch](https://distrowatch.com)  
[GitHub - Awesome Linux Software](https://github.com/luong-komorebi/Awesome-Linux-Software)  
[GitHub - Linux Guide](https://github.com/mikeroyal/Linux-Guide)  
[GNU](https://www.gnu.org/)  
[Linux manual page](https://man7.org/linux/man-pages/index.html)  
[Manual pages from Arch Linux packages](https://man.archlinux.org/)  
[The Linux Kernel Archives](https://www.kernel.org/)  
[Wikipedia - List of Linux distributions](https://en.wikipedia.org/wiki/List_of_Linux_distributions)


## Distribuições Linux
Não existe distribuição Linux perfeita, sempre haverá uma distribuição para um caso de uso específico.

Por exemplo, minha percepção atual é a seguinte:  
[Alpine Linux](https://www.alpinelinux.org/) para containers e testes;  
[Arch Linux](https://archlinux.org/) como desktop de uso pessoal;  
[Fedora](https://fedoraproject.org/) como *workstation*, desktop de uso pessoal, live USB e máquinas virtuais;  
[Tails](https://tails.net/) se o foco for privacidade;  
[Whonix](https://www.whonix.org/) se o foco for privacidade.

Para cada necessidade sempre haverá uma distribuição diferente.  
Teste diferentes distribuições até encontrar a que mais se adapte ao seu uso.


## Arch Linux
O [Arch Linux](https://wiki.archlinux.org/title/Arch_Linux) se destaca por sua filosofia minimalista, por permitir que o usuário crie o seu sistema e por ter uma das [wikis](https://wiki.archlinux.org) mais completas em toda internet.

### Instalação
Evite tutoriais, seu dispositivo e uso certamente são diferentes de outros.  
Utilize a [documentação de instalação](https://wiki.archlinux.org/title/installation_guide) ou o [archinstall](https://wiki.archlinux.org/title/Archinstall) para uma instalação guiada.  
No meu caso, seguindo a documentação, precisei alterar as seguintes etapas:

#### Particionamento do dispositivo
Usando o [GPT fdisk](https://wiki.archlinux.org/title/GPT_fdisk), gosto de seguir o layout abaixo.
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

O layout utilizado para os subvolumes é o mesmo usado pelo `archinstall` e sugerido pelo [Snapper](https://wiki.archlinux.org/title/Snapper#Suggested_filesystem_layout).

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

No momento de instalação dos pacotes bases, instale também o [microcode](https://wiki.archlinux.org/title/Microcode) correspondente.

`pacstrap -K /mnt amd-ucode base linux linux-firmware nano networkmanager`

#### Localização
Edite o `/etc/locale.gen` removendo o comentário de `en_US.UTF-8 UTF-8` e `pt_BR.UTF-8 UTF-8`.  
Execute o comando `locale-gen`.  

`echo "LANG=en_US.UTF-8" >> /etc/locale.conf`  
`echo "KEYMAP=br-abnt2" >> /etc/vconsole.conf`  
Altere as configurações de formato utilizando o ambiente gráfico e após a instalação do sistema.

#### Host
```plaintext
#/etc/hosts

127.0.0.1     localhost archlinux
::1           localhost archlinux
```

#### Boot loader
Instale o `systemd-boot`:  
`bootctl install`

Configure o `loader.conf` conforme o exemplo [loader configuration](https://wiki.archlinux.org/title/Systemd-boot#Loader_configuration).  
Adicione a *entry* `arch.conf` conforme o exemplo [adding loaders](https://wiki.archlinux.org/title/Systemd-boot#Adding_loaders).

Ao usar algum microcode, como a `amd-ucode.img`, deve-se sempre referenciá-la antes do `initrd` principal.

Para utilizar um subvolume como um ponto de montagem, faça o apontamento no *loader*. Exemplo [mounting subvolume as root](https://wiki.archlinux.org/title/Btrfs#Mounting_subvolume_as_root).

```sh
# arch.conf
title   Arch Linux
linux   /vmlinuz-linux
initrd  /amd-ucode.img
initrd  /initramfs-linux.img
options root=UUID=XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX rootflags=subvol=@ rw

# UUID, PARTUUID ou semelhante são coisas diferentes.
#
# É possível consultar as identificações de /dev/root_partition usando:
# ls -l /dev/disk/by-*
# blkid /dev/root_partition
#
# Também é possível usar diretamente o caminho do dispositivo:
# options root=/dev/root_partition rw
```

Valide a configuração:  
`bootclt list`

#### Pacotes básicos
Pacotes que gosto de usar para uma boa experiência de uso do sistema.

```text
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
smartmontools
sudo
wget
which
wireplumber
xdg-utils
zip
zsh
```

Habilite os serviços necessários:  
`systemctl enable gdm.service`  
`systemctl enable NetworkManager`

#### Usuário
O root deve ser usado apenas em tarefas específicas ao `root`.  

`useradd -m -G wheel -s /bin/bash usuario`  
`passwd usuario`

Também é necessário configurar o grupo `wheel` utilizando o `visudo` para [permitir que usuários usem o sudo](https://wiki.archlinux.org/title/sudo#Example_entries).

Nesse ponto, já é possível realizar o `umount -R /mnt` e reiniciar o sistema.

#### Drivers
Como usuário [AMDGPU](https://wiki.archlinux.org/title/AMDGPU) leia o artigo e entenda o que é necessário.  
Configure também o [lm_sensors](https://wiki.archlinux.org/title/Lm_sensors).

### Configurações adicionais
#### GNOME
Caso necessário, altere o funcionamento do [clipboard](https://wiki.archlinux.org/title/Clipboard).  

[Criar documentos com o menu do clique direito](https://wiki.archlinux.org/title/GNOME/Files#Create_a_new_document_from_the_right-click_menu).

Instale o `Extension Manager` pelo GNOME Software (Flathub).

Instale e/ou ative as extensões:  
[Cliboard Indicator](https://extensions.gnome.org/extension/779/clipboard-indicator/)  
[Removable Drive Menu](https://extensions.gnome.org/extension/7/removable-drive-menu/)  
[System Monitor](https://extensions.gnome.org/extension/6807/system-monitor/)

#### ZSH
Instale o [Oh My Zsh](https://github.com/ohmyzsh/ohmyzsh) e os seguintes plugins:  
[Powerlevel10k](https://github.com/romkatv/powerlevel10k)  
[zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)  
[zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting)

Ao usar o `Oh My Zsh` sempre realize a instalação de temas e *plugins* por meio dele.  
Crie um script para atualizar periodicamente os temas e *plugins*, basta um `git pull` nos diretórios em `~/.oh-my-zsh/custom/plugins/` e `~/.oh-my-zsh/custom/themes/`.

O comportamento padrão de sugestão do após colar é bastante irritante, ele pode ser removido da seguinte maneira:

```bash
plugins=(...)

source $ZSH/oh-my-zsh.sh

# User configuration
ZSH_AUTOSUGGEST_CLEAR_WIDGETS+=(bracketed-paste)
```

#### nano
Use a instrução `echo "set linenumbers" > ~/.nanorc` para exibir o número de linhas.

#### pacman
```bash
du -hs /var/cache/pacman/pkg/ # Verifica o espaço consumido pelo cache dos pacotes

pacman -Sc                    # Remove do cache versões antigas dos pacotes instalados
pacman -Scc                   # Remove todos os pacotes do cache

pacman -Qtdq                  # Lista pacotes instalados como dependências que não são requeridos por nenhum outro pacote
pacman -R $(pacman -Qtdq)     # Remove os pacotes listados acima
```


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


