## Sistema de arquivos
### B-Tree File System
Btrfs (B-Tree File System) é um sistema de arquivos cujo funcionamento é baseado em COW.  
Nem todos os [recursos do Btrfs](https://btrfs.readthedocs.io/en/latest/Status.html) são considerados estáveis, recomenda-se utilizar os com status de estável.

#### COW (Copy-On-Write)
Quando um arquivo é modificado o bloco de dados existente no dispositivo não é alterado, apenas as alterações são gravadas em novos blocos com referências ao arquivo original.

Habilitado por padrão, corresponde a opção `datacow` no *mount* do subvolume.

#### Compressão
Btrfs suporta compressão, o que ajuda a aumentar significativamente a vida útil de dispositivos SSD.  
As opções de compressão são `ZLIB`, `LZO` e `ZSTD`, sendo `ZSTD` a recomendada.

Pode ser habilitada utilizando a opção `compress=zstd` no *mount* do subvolume.

Para aplicar desfragmentação e compactar arquivos existentes:  
`btrfs filesystem defragment -r -v -c<tipo> /home`  
`btrfs filesystem defragment -r -v -czstd:3 /home`  

#### TRIM
TRIM SSD é uma tecnologia em SSD para permitir que sistema operacional informe ao SSD quais blocos de dados não estão mais em uso e podem ser apagados.  
O TRIM é crucial para manter o desempenho e a durabilidade do SSD.

Habilitado por padrão no Btrfs caso o dispositivo suporte, corresponde a opção `discard=async` no *mount* do subvolume.

```bash
lsblk --discard /dev/sdX
# DISC-GRAN   Tamanho mínimo de dados que podem ser descartados
# DISC-MAX    Tamanho máximo de dados que podem ser descartados de uma vez
# Se ambas forem diferentes de 0B, o SSD suporta TRIM

# Executa o TRIM
sudo fstrim -v /
```

#### Subvolume
Pode-se considerar um subvolume como um namespace de arquivo POSIX, ou seja, um delimitador abstrato, uma espécie de container, para os arquivos que ele armazena.  
Subvolumes podem ser criados em qualquer lugar dentro da hierarquia do sistema de arquivos.  
Caso um subvolume tenha o ponto de montagem em um diretório dentro do ponto de montagem de outro subvolume, ele funcionará de forma completamente independente.

Exemplo:

```plaintext
partição    subvolume     ponto de montagem
/dev/vda2   /@            /
/dev/vda2   /@home        /home               Independente de @
/dev/vda2   /@home/xyz    /home/xyz           Independente de @ e @home 
/dev/vda2   /@.snapshot   /.snapshot          Independente de @
```

Se o subvolume representar algo referente ao sistema, seu gerenciamento deve ser feito por um LiveUSB.

Em `/etc/fstab` é possível notar que um único UUID é usado por todos subvolumes montados em uma partição.  
Já quanto as partições, elas não compartilham o mesmo UUID por serem dispositivos lógicos ou físicos diferentes.

Da mesma forma que os subvolumes compartilham o UUID, eles também compartilham o armazenamento.  
Sistemas que compartilham espaço de armazenamento, podem ter problemas caso algum ponto do sistema fique sobrecarregado, como `/var/log`.

O Btrfs suporta o uso de quotas, permitindo que o armazenamento seja reservado por subvolume.

Exemplo de comandos para gerenciar subvolumes:

```bash
btrfs subvolume list /
btrfs subvolume show /home/xyz
btrfs subvolume create /home/xyz
btrfs subvolume delete /home/xyz

btrfs filesystem df /dev/mount_point      # Uso do espaço em nível lógico no sistema de arquivos Btrfs
btrfs filesystem usage /dev/mount_point   # Informações detalhadas do uso de espaço

# Monta o subvolume raiz do Btrfs (sempre será o 5)
# O subvolume raiz armazena todos os subvolumes
# Usado para gerenciar a estrutura e subvolumes do Btrfs
mount -o subvolid=5 -t btrfs /dev/device /mnt

# Monta um subvolume específico
# O subvolume "@" normalmente representa a raiz do sistema "/"
# Usado para montar um subvolume em um ponto de montagem
mount -o subvol=@ ...
```

#### Snapshot
Devido ao CoW, quando uma *snapshot* é criada, o Btrfs cria um ponto de referência para os dados e metadados existentes em um subvolume, sem duplicá-los.  
Apenas o que for alterado em relação ao *snapshot* será gravado.

Uma *snapshot* é uma cópia de um subvolume em um momento específico, dessa forma, uma *snapshot* também é considerada um subvolume.  
*Snapshots* se diferem de subvolumes ao propósito, uma *snapshot* tem por objetivo preservar um ponto no tempo de um subvolume.

Restaurando uma *snapshot* da raiz do sistema:

```bash
btrfs subvolume snapshot / /.snapshots/@yy-mm-dd  # Não use a opção -r nesse contexto

mount -o subvolid=5 -t btrfs /dev/device /mnt     # LiveUSB

btrfs subvolume list /mnt

mv /mnt/@ /mnt/@backup
mv /mnt/@.snapshots/@yy-mm-dd /mnt/@

btrfs subvolume show /mnt/@
btrfs subvolume set-default numero-id /mnt

umount /mnt
reboot

btrfs subvolume list /                            # Opcional
btrfs subvolume show -r numero-id /               # Opcional
btrfs subvolume delete -i numero-id /             # Opcional

btrfs balance start /                             # Opcional
```

Criando um subvolume e restaurando uma *snapshot* em um diretório não essencial do sistema:

```bash
btrfs sub create /my/dir                    # Cria um subvolume qualquer
btrfs sub list /
btrfs sub snaps -r /my/dir /.snapshots/snap # -r, read-only é ideal para backups ou versões imutáveis

btrfs sub d /my/dir
btrfs sub snaps /.snapshots/snap /my/dir    # Restaura a snapshoft subvolume
```

#### Aliases
O programa `btrfs` conta com diversos alias.  
Exemplos:

```plaintext
btrfs subvolume list          btrfs sub l
btrfs subvolume snapshot      btrfs sub snaps
btrfs subvolume delete        btrfs sub d
btrfs subvolume create        btrfs sub c
btrfs subvolume show          btrfs sub sh
btrfs subvolume set-default   btrfs sub set

btrfs filesystem show   btrfs fi sh
btrfs filesystem df     btrfs fi df
btrfs filesystem usage  btrfs fi us
btrfs filesystem label  btrfs fi lab
```
