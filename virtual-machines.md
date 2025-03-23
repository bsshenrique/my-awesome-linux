# Máquinas Virtuais
## Introdução
Um entendimento prévio do tema é fundamental.

### BIOS - Basic Input/Output System
*Firmware* armazenado em um chip na placa-mãe, responsável por inicializar o *hardware* e carregar o sistema operacional.  
Executa o POST (Power-On Self-Test) antes de buscar o *bootloader* no dispositivo de armazenamento.  
O BIOS tem uma interface de gerenciamento simples.  
Utiliza o esquema de particionamento MBR.

### MBR - Master Boot Record
Esquema de particionamento de discos com o limite de 2TB por partição e no máximo 4 partições primárias (ou 3 primárias + 1 estendida).

### UEFI - Unified Extensible Firmware Interface
Evolução do BIOS.  
Conta com uma interface gráfica mais avançada, Secure Boot e tempos de *boot* mais rápidos.  
Pode carregar diretamente um arquivo executável EFI localizado na partição ESP (EFI System Partition), sem precisar de um *bootloader* tradicional.  
Utiliza o esquema de particionamento GPT.

### GPT - GUID Partition Table
Esquema de particionamento com suporte a partições maiores do que 2TB.

### Secure Boot
Recurso de segurança da UEFI para impedir a execução de software não autorizado durante a inicialização do sistema.  
Verifica assinaturas digitais dos *bootloaders* e *drivers*.

### TPM - Trusted Platform Module
Chip de segurança presente em algumas placas-mãe usado para armazenar chaves criptográficas e proteger contra ataques de software e roubo de credenciais.  
Amplamente utilizado para criptografia de disco (como o BitLocker no Windows) e autenticação segura.

### Hypervisor
*Hypervisor* é um software capaz de gerenciar, segmentar e alocar recursos de hardware para VMs.

*Hypervisors* do **tipo 1** são chamados de nativos ou *bare metal*.  
São executados diretamente no hardware e têm a capacidade de gerenciar diversos sistemas operacionais.  
Sua arquitetura é basicamente `hardware > hypervisor > VM`.  
Exemplos: KVM, QEMU KVM.

Já os de **tipo 2** são chamados de *hosted*.  
Funciona como um software instalado sobre um sistema operacional.  
Sua arquitetura é basicamente `hardware > OS > hypervisor > VM`.  
Exemplos: QEMU.

### KVM - Kernel-based Virtual Machine
KVM é um módulo do *kernel* Linux capaz de fazer com que o sistema se comporte como um *hypervisor bare metal*.  
Possibilita o uso de máquinas virtuais com alto desempenho, aproveitando extensões de virtualização de hardware (VT-x e AMD-V).  

### QEMU - Quick Emulator
Funciona como um emulador (virtualização *hosted*) de diferentes sistemas operacionais e arquiteturas de CPU.  
Quando usado com KVM proporciona virtualização *bare metal*, permitindo um desempenho próximo do nativo.  

### Hardware-assisted virtualization
Instrução presente em processadores para permitir virtualização.  
Possibilita que um *hypervisor* (como o KVM) acesse diretamente as instruções do processador, sem precisar da emulação, dessa forma permite que uma VM acesse diretamente o processador.  
Processadores AMD: AMD-V, AMD Virtualization (antes chamado de SVM - Secure Virtual Machine)
Processadores Intel: VT-x

### I/O MMU virtualization
Permite que as VMs tenham acesso direto a dispositivos de hardware, como GPU, controladores de rede e de armazenamento.  
É essencial para configurar o PCI Passthrough em ambientes de virtualização.  

Processadores AMD: AMD-Vi (antes chamado de IOMMU)
Processadores Intel: VT-d


## Instalação
### Requisitos
Certifique-se de que sua CPU suporta virtualização e que ela está habilitada no UEFI.

Confira se a virtualização está disponível:  
`lscpu | grep -i virtualization`

Confira se o módulo `kvm` foi carregado corretamente:  
`lsmod | grep kvm`

### Pacotes necessários
Considerando o uso do Arch Linux, vamos aos nomes de alguns pacotes.

```plaintext
dnsmasq         Usado pelo virtual network switch (por padrão o virbr0) para disponibilizar NAT, DHCP e DNS para as VMs
edk2-ovmf       EFI Development Kit 2 Open Virtual Machine Firmware, possibilita o uso de UEFI em VMs
libvirt         Backend/API de gerenciamento unificado entre o hypervisor e cliente (virt-manager, virsh...)
- virsh         Cliente/interface gerenciamento do hypervisor por CLI
qemu-desktop    Virtualização da arquitetura x86_64
qemu-full       Virtualização de diversas arquiteturas
spice-vdagent   Agente SPICE (guest)
swtpm           Software TPM, emulador TPM para VMs
virt-manager    Cliente/interface gerenciamento do hypervisor por GUI
virt-viewer     Cliente SPICE/VNC (host)
```

#### SPICE
SPICE, Simple Protocol for Independent Computing Environments, é um protocolo usado para acesso remoto à VMs.  
Por integrar recursos entre *host* e *guest*, também oferece uma experiência mais fluida na VM.

Embora melhore o desempenho significativamente, é o suficiente apenas para tarefas básicas.  
Tarefas pesadas como jogos modernos exigem o uso de uma GPU dedicada via PCI passthrough.

Para que o SPICE funcione, o pacote `spice-vdagent`deve ser instalado no *guest*.

### Instalação
Instale os seguintes pacotes:  
`sudo pacman -S dnsmasq edk2-ovmf libvirt qemu-desktop swtpm virt-manager virt-viewer`

`sudo systemctl enable libvirtd.service`  
`sudo systemctl start libvirtd.service`  
`sudo systemctl status libvirtd.service`

`sudo usermod -aG libvirt $USER`  
`reboot`

### Configuração
No setup de criação da VM siga com modo padrão (NAT) utilizando [Link-level address caveat](https://wiki.archlinux.org/title/QEMU#Link-level_address_caveat).  
Para entender sobre os modos de operação e como funciona o *virtual network switch* confira o artigo [Virtual Networking](https://wiki.libvirt.org/VirtualNetworking.html).

Ao usar o modo NAT, certifique-se de que o `iptables` foi configurado:
```plaintext
# /etc/libvirt/network.conf`
firewall_backend = "iptables"
```

```plaintext
Overview
Hypervisor: KVM
Chipset:    Q35
Firmware:   UEFI  (OVMF_CODE.4m.fd)
                  (se precisar de secure boot: OVMF_CODE.secboot.4m.fd)

Melhora o desempenho no guest
Display
Type:         Spice server
Listen type:  None
OpenGL:       true (auto)

Video
Model:            Virtio
3D acceleration:  true

Habilita o shared clipboard
Channel
Name:         com.redhat.spice.0
Device type:  Spice agent (spicevmc)
```

### Dicas
```bash
# (Gnome) Histórico de ISOs
gsettings get org.virt-manager.virt-manager.urls isos

# (Gnome) Limpa o histórico de ISOs
gsettings reset org.virt-manager.virt-manager.urls isos
```
