# Máquinas Virtuais
> [!NOTE]
> Todo o conteúdo do artigo é baseado em QEMU, KVM e `libvirt`.


## Conceitos
Um entendimento prévio do tema é fundamental.

### BIOS - Basic Input/Output System
*Firmware* armazenado em um chip na placa-mãe.  
*Schema* de particionamento: MBR - Master Boot Record.

Realiza funções como inicializar o hardware, executa o POST (Power-On Self-Test) e carregar o bootloader na MBR.

### UEFI - Unified Extensible Firmware Interface
*Firmware* mais moderno, rápido e é considerado o substituto do BIOS.  
*Schema* de particionamento: GPT - GUID Partition Table.

#### ESP - EFI System Partition
Partição especial obrigatória em sistemas que utilizam o *firmware* UEFI.  
Armazena arquivos de bootloader em formato `.efi`.  
Exemplo: `EFI/BOOT/BOOTX64.EFI`

#### Secure Boot
recurso de segurança em sistemas com *firmware* UEFI.  
Garante que durante a inicialização, apenas softwares confiáveis (assinados digitalmente) sejam carregados pelo *firmware*.  
Seu propósito é evitar que *malwares* como *bootkits* e *rootkits* se infiltrem no processo de *boot* antes do sistema operacional assumir o controle.

### TPM - Trusted Platform Module
Chip com o propósito de segurança usado para armazenar chaves criptográficas.  
Utilizado para criptografia de disco (como o BitLocker no Windows) e autenticação segura.

dTPM: discrete TPM, chip físico na placa-mãe.  
fTPM: firmware Trusted Platform Module, para AMD.  
PTT: Platform Trust Technology, para Intel.

### Hardware-assisted virtualization
Instrução presente em processadores para permitir virtualização.  
Possibilita que um *hypervisor* (como o KVM) acesse diretamente o processador, sem precisar da emulação.  
Processadores AMD: AMD-V, AMD Virtualization (antes chamado de SVM - Secure Virtual Machine).  
Processadores Intel: VT-x.

### I/O MMU virtualization
Permite que as VMs tenham acesso direto a dispositivos de hardware, como GPU, controladores de rede e de armazenamento.  
É essencial para configurar o PCI Passthrough em ambientes de virtualização.  
Processadores AMD: AMD-Vi (antes chamado de IOMMU).  
Processadores Intel: VT-d.

### Hypervisor
*Hypervisor* é um software capaz de gerenciar, segmentar e alocar recursos de hardware para VMs.

*Hypervisors* do **tipo 1** são chamados de nativos ou *bare metal*.  
São executados diretamente no hardware e têm a capacidade de gerenciar diversos sistemas operacionais.  
Arquitetura: `hardware > hypervisor > VM`.  
Exemplos: KVM, QEMU KVM.

Já os de **tipo 2** são chamados de *hosted*.  
Funciona como um software instalado sobre um sistema operacional.  
Arquitetura: `hardware > OS > hypervisor > VM`.  
Exemplos: QEMU.

#### KVM - Kernel-based Virtual Machine
KVM é um módulo do *kernel* Linux capaz fornecer virtualização acelerada por *hardware* (VT-x e AMD-V).  
Por operar diretamente sobre o *kernel*, é um *hypervisor* do tipo *bare metal*.

#### QEMU - Quick Emulator
Funciona como um emulador (virtualização *hosted*) de diferentes sistemas operacionais e arquiteturas de CPU.  
Quando usado com KVM proporciona virtualização do tipo *bare metal*.


## Requisitos
Certifique-se de que sua CPU suporta virtualização e que ela está habilitada na UEFI.

Confira se a virtualização está disponível:  
`lscpu | grep -i virtualization`

Confira se o módulo `kvm` foi carregado corretamente:  
`lsmod | grep kvm`


## Instalação
Antes da instalação, tenha ciência do propósito do propósito dos seguintes pacotes:

```plaintext
dnsmasq         Usado pelo virtual network switch (por padrão o virbr0) para disponibilizar NAT, DHCP e DNS para as VMs
edk2-ovmf       EFI Development Kit 2 Open Virtual Machine Firmware, possibilita o uso de UEFI em VMs
libvirt         Backend/API de gerenciamento unificado entre o hypervisor e cliente (virt-manager, virsh...)
- virsh         Cliente/interface gerenciamento do hypervisor por CLI
qemu-desktop    Virtualização da arquitetura x86_64
qemu-full       Virtualização de diversas arquiteturas
spice-vdagent   Agente SPICE (guest)
swtpm           Software TPM, emulador TPM para VMs
virt-firmware   Ferramentas para manipular imagens de firmware edk2
virt-manager    Cliente/interface gerenciamento do hypervisor por GUI
virt-viewer     Cliente SPICE/VNC (host)
```

No *host*, instale os seguintes pacotes:

```bash
sudo pacman -S dnsmasq edk2-ovmf libvirt qemu-desktop swtpm virt-manager virt-viewer

sudo systemctl enable libvirtd.service
sudo systemctl start libvirtd.service
sudo systemctl status libvirtd.service

sudo usermod -aG libvirt $USER
reboot
```

No *guest*, instale o `spice-vdagent`.

SPICE, Simple Protocol for Independent Computing Environments, é um protocolo usado para acesso remoto à VMs.  
Por integrar recursos entre *host* e *guest*, também oferece uma experiência mais fluida na VM.

Embora melhore o desempenho significativamente, é o suficiente apenas para tarefas básicas.  
Tarefas pesadas como jogos modernos exigem o uso de uma GPU dedicada via PCI *passthrough*.


## Configuração
### iptables
No setup de criação da VM siga com modo padrão (NAT) utilizando [Link-level address caveat](https://wiki.archlinux.org/title/QEMU#Link-level_address_caveat).  
Para entender sobre os modos de operação e como funciona o *virtual network switch* confira o artigo [Virtual Networking](https://wiki.libvirt.org/VirtualNetworking.html).

Se estiver usando o `iptables` ou `iptables-nft` e optar por NAT, certifique-se de que o `iptables` está configurado no `libvirt`:

```plaintext
# /etc/libvirt/network.conf`
firewall_backend = "iptables"
```

### Secure Boot
Conforme mencionado em [Enabling Secure Boot](https://wiki.archlinux.org/title/QEMU#Enabling_Secure_Boot), o Arch Linux não conta com o seu próprio `/usr/share/edk2/x64/OVMF_VARS.secboot.4m.fd` contendo chaves `pre-enrolled`.

Para contornar esse impedimento, siga as instruções abaixo.

Realize o download da versão `noarch` mais recente do pacote [edk2-ovmf](https://packages.fedoraproject.org/pkgs/edk2/edk2-ovmf/).  
Em seguida realize as instruções:

```bash
rpm2cpio edk2-ovmf-*.*.noarch.rpm | cpio -id

sudo qemu-img convert -O raw -f qcow2 usr/share/edk2/ovmf/OVMF_VARS_4M.secboot.qcow2 OVMF_VARS.secboot.4m.fd

sudo mv OVMF_VARS.secboot.4m.fd /usr/share/edk2/x64/

# Altere vm_name para o nome da VM
virt-fw-vars
  --input /usr/share/edk2/x64/OVMF_VARS.4m.fd
  --output /var/lib/libvirt/qemu/nvram/vm_name_SECURE_VARS.fd
  --secure-boot
  --enroll-redhat
```

Certifique-se de que o XML de configuração da VM esteja conforme o exemplo:

```xml
<os firmware="efi">
  <loader readonly="yes" secure="yes" type="pflash">/usr/share/edk2/x64/OVMF_CODE.secboot.4m.fd</loader>
  <nvram template="/usr/share/edk2/x64/OVMF_VARS.4m.fd">/var/lib/libvirt/qemu/nvram/{vm-name}_SECURE_VARS.fd</nvram>
</os>
```

### Setup
```plaintext
# Overview
Hypervisor:       KVM
Chipset:          Q35
Firmware:         UEFI (edk2/x64/OVMF_CODE.secboot.4m.fd)

# Melhora o desempenho no guest
# Display
Type:             Spice server
Listen type:      None
OpenGL:           true (auto)

# Video
Model:            Virtio
3D acceleration:  true

# Habilita o shared clipboard
# Channel
Name:             com.redhat.spice.0
Device type:      Spice agent (spicevmc)
```

### Dicas
```bash
# (Gnome) Histórico de ISOs
gsettings get org.virt-manager.virt-manager.urls isos

# (Gnome) Limpa o histórico de ISOs
gsettings reset org.virt-manager.virt-manager.urls isos
```
