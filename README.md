# My Awesome Linux
![GitHub last commit (branch)](https://img.shields.io/github/last-commit/bsshenrique/my-awesome-linux/main)

**My Awesome Linux** é um repositório pessoal descrevendo o que considero em um ambiente Desktop Linux.  

## Tópicos
- [Comunidade Linux](#comunidade)
- [Distribuições](#distribuições)
  - [Arch Linux](#arch-linux)
    - [Instalação](#instalação)

## Comunidade
Sites úteis criados pela comunidade Linux.

**[Awesome Linux Software](https://github.com/luong-komorebi/Awesome-Linux-Software)**  
**[Diolinux](https://diolinux.com.br/)**  
**[DistroSea](https://distrosea.com/)**  
**[DistroWatch](https://distrowatch.com)**  
**[GuiaFoca](https://www.guiafoca.org/)**  
**[Linux Brasil](https://www.reddit.com/r/linuxbrasil/)**  
**[Linux Guide](https://github.com/mikeroyal/Linux-Guide)**  
**[List of Linux distributions](https://en.wikipedia.org/wiki/List_of_Linux_distributions)**.

## Distribuições
Não existe distribuição Linux perfeita, sempre haverá uma distribuição para um caso de uso específico. Por exemplo, a minha percepção atual é a seguinte:

[Alpine Linux](https://www.alpinelinux.org/) para containers e testes;  
[Arch Linux](https://archlinux.org/) como desktop para uso pessoal;  
[Tails](https://tails.net/) se o foco for privacidade;  
[Ubuntu](https://ubuntu.com/) e [Fedora](https://fedoraproject.org/) muito úteis para live USB e máquinas virtuais.

Para cada necessidade sempre haverá uma distribuição diferente.  
O mais importante é entender que o meu propósito não é o mesmo que o seu, teste diferentes distribuições até encontrar a que mais se adapte ao propósito buscado.  

### Arch Linux
Quanto mais simples, melhor, e é por isso que o [Arch Linux](https://wiki.archlinux.org/title/Arch_Linux) é um sistema operacional que me chama a atenção.  
O Arch Linux tem como propósito simplicidade e uso apenas do essencial.

Pode parecer estranho mencionar estabilidade em [rolling release](https://wiki.archlinux.org/title/system_maintenance#Partial_upgrades_are_unsupported), mas em minha experiência nunca tive problemas e por mais controverso que pareça, sempre tive problemas com distribuições Linux que oferecem o modelo de distribuição por "major updates", principalmente Ubuntu.  


Se você ainda pensa "o Linux deu problema e agora só formatando", tire isso da cabeça.  
Além da [wiki](https://wiki.archlinux.org) do Arch Linux ser a documentação mais completa que provavelmente existe em toda internet, também existem formas de resolver problemas, como por exemplo, o uso de snapshots oferecidas pelo [Btrfs](https://wiki.archlinux.org/title/btrfs).

#### Instalação
Evite seguir tutoriais, seu dispositivo e necessidades certamente são diferentes de outros.  
Use como base a [documentação de instalação](https://wiki.archlinux.org/title/installation_guide) e modifique o que for necessário para o seu uso.  

No meu caso, seguindo o tutorial, precisei alterar as seguintes etapas:  

- **Particionamento do disco**

Com o [GPT fdisk](https://wiki.archlinux.org/title/GPT_fdisk), o disco foi preparado com o seguinte layout:

| Partição | Setor              | Tipo | Montagem  | Formato |
| :------: | :----------------: | :--: | :-------: | :-----: |
| 1        | default à +512M    | EF00 | /mnt/boot | FAT32   |
| 2        | default à +2G      | 8200 | SWAP      | -       |
| 3        | default à default  | 8304 | /mnt      | ext4    |

- **Pacotes essenciais**

`base dhcpcd linux-lts linux-firmware nano`

- **Localização**

Sistema operacional configurado em inglês e os formatos em português:

```bash
# /etc/locale.gen

en_US.UTF-8 UTF-8
pt_BR.UTF-8 UTF-8
```

`locale-gen`.

- **Microcode**

[Microcode](https://wiki.archlinux.org/title/Microcode) correspondente ao dispositivo.

- **Boot loader**

Apenas o  básico do [systemd-boot](https://wiki.archlinux.org/title/Systemd-boot) já é o suficiente para que tudo funcione perfeitamente.  
O deve `loader.conf` conforme sugestão e um único loader nomeado de `arch.conf` já é o suficiente.

```bash
# arch.conf

title   Arch Linux
linux   /vmlinuz-linux-lts
initrd  /amd-ucode.img
initrd  /initramfs-linux-lts.img
# Cuidado.
# Se for definir o root por UUID, PARTUUI ou semelhante, saiba que são coisas diferentes.
# options root=UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx rw
# options root=PARTUUI=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx rw
#
# Consulte com uma das formas:
# /dev/disk/by-*
# $ blkid /dev/disco
#
# Ou use diretamente o caminho do disco
options root=/dev/particao rw
```

Novamente, cuidado.  
Entenda o que é [UEFI](https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface) e [Secure Boot](https://wiki.archlinux.org/title/Unified_Extensible_Firmware_Interface/Secure_Boot).  

A forma mais fácil que conheço para preparar o Arch Linux em dispositivos com Secure Boot é desabilitar o Secure Boot, em seguida limpar as chaves já configuradas e finalmente criar e assinar as chaves utilizando o `sbctl`.

- **Usuário**

Antes de instalar um ambiente desktop, prefiro criar o meu próprio usuário.  
Não gosto de utilizar o root em ambiente gráfico, em minha concepção o root só deve ser usado em tarefas específicas ao root.  

`useradd -m -G wheel -s /bin/bash usuario`  
`passwd usuario`

- **Pacotes básicos**

Pacotes para uma boa experiência de uso do sistema.

```text
base-devel
btop
firefox
gnome
less
networkmanager
noto-fonts
noto-fonts-cjk
noto-fonts-emoji
noto-fonts-extra
pipewire
pipewire-alsa
pipewire-audio
pipewire-jack
pipewire-pulse
sudo
ttf-hack
wget
which
wireplumber
zip
zsh
```

`systemctl enable gdm.service`  
`systemctl enable NetworkManager`

Também é necessário configurar o grupo `wheel` utilizando o `visudo` para [permitir que usuários do grupo usem o sudo](https://wiki.archlinux.org/title/sudo#Example_entries).

- **Pacotes extras**

Pacotes para uma boa experiência de trabalho e lazer.

```text
git
neofetch
```

Utilizar o [Oh My Zsh](https://github.com/ohmyzsh/ohmyzsh) com os plugins [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions), [zsh-syntax-highlighting](https://github.com/zsh-users/zsh-syntax-highlighting) e o tema [Powerlevel10k](https://github.com/romkatv/powerlevel10k) certamente vão oferecer uma boa produtividade.

Se o seu dispositivo ter uma placa gráfica da AMD, leia o guia [AMDGPU](https://wiki.archlinux.org/title/AMDGPU).  
Provavelmente você irá instalar os seguintes pacotes:
```text
lib32-vulkan-radeon
lib32-libva-mesa-driver
libva-mesa-driver
mesa
vulkan-radeon
xf86-video-amdgpu
```

- **Configurações adicionais**

O guia [selection](https://wiki.archlinux.org/title/clipboard#Selections) pode ser muito útil aos usuários do `kgx`.

Para usuários do GNOME, pode ser bem útil [criar documentos com o menu do clique direito](https://wiki.archlinux.org/title/GNOME/Files#Create_a_new_document_from_the_right-click_menu).


