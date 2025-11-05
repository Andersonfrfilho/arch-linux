# arch-linux

# Partições

##################
# Nutella / básico 500GBSSD - 32GBRaM
##################
/ -> raiz do projeto (100 - G)
/home -> pastas e arquivos do usuario (340 G)
/swap -> troca e apoio a memoria ram caso vc enxa ela ele utiliza o swap (64 G)

###################
# Intermeduario    500GBSSD - 32GBRaM - usuarios
###################
Informação de Legacy ou UEFI encontra na setup da placa mãe
/boot (para legacy)->primeiro setor do disco, tem que ficar em primeiro pra ficar mais rapido (500 M)
/efi  (para UEFI) -> devices mais novos      (500 M)
/                                            (70 G)
/usr  |--> binarios dos softwares            (30 G)
/home |--> arquivos do usuario               (280 G)
/opt  |--> optionais (software de terceiros) (30 G)
/tmp  |--> arquivos temporarios              (3 G)
swap  |--> troca de mem Ram para SSD         (64 G)

###################
# Avançado (configuração para servidor)
###################
/boot (para legacy)->primeiro setor do disco, tem que ficar em primeiro pra ficar mais rapido
/efi  (para UEFI) -> devices mais novos

/     
/usr  |--> bibliotecas, módulos apoios do sistemas python
/tmp  |--> Arquvios temporarios
/etc  |--> ssh / mysql / ngnix arquivos e configurações globais
/var  |--> aquivos variaveis - que precisa ser trocado logs, armazenados aqui logs do mysql..., apache descarregamento de alguma configuração
/srv  |--> link symbolicos diretoria fixo, configurações servidor como do apache estarão aqui
/home |--> usuarios para trabalhar no ambiente, cada usuario tem seu home
/opt  |--> programaças opcionais software de terceiros (programas que tem estruturas proprias (arquivo de configurações etc

selecione a medium isntalation
configure o teclado caso utilize  abnt2
1 - rode o comando caso utilize loadkeys br-abnt2
2 - cgdisk | cfdisk
3 verifique o hd selecione ele e apague
4 escreva as partições
| Partição           | Tamanho sugerido       | Sistema de arquivos | Montagem    | Observações                                  |
| ------------------ | ---------------------- | ------------------- | ----------- | -------------------------------------------- |
| **EFI System**     | 512 MB                 | FAT32               | `/boot/efi` | Necessária para UEFI                         |
| **Swap**           | 4 GB a 8 GB            | swap                | —           | Se tiver 16 GB+ de RAM, 4 GB é suficiente    |
| **Root (`/`)**     | 40 GB                  | ext4                | `/`         | Contém o sistema e pacotes                   |
| **Home (`/home`)** | **restante (~190 GB)** | ext4                | `/home`     | Seus projetos, containers, bancos locais etc |
5 - formatar as partições
  - formatar a partição UEFI
     - mkfs.fat -F32 /dev/)(verificar a partição do EFI
  - formatar as ext4
    - mkfs.ext4 /dev/sda2
    - mkfs.ext4 /dev/sda3
  - criaçao do swap
    - mkswap /dev/sda3
montar a partição raiz
  1 - lista lsblk
  2 - mounte a partição raiz
      - /dev/sda2
      - mount /dev/sda2 /mnt
    - liste para confirmar
      - /mnt
  3 - criar as partições
    - mkdir -p /mnt/boot/efi
       
⚙️ OBJETIVO

Você quer:

desmontar tudo o que está montado em /mnt

reativar e montar corretamente:

EFI → /mnt/boot/efi

ROOT (/) → /mnt

HOME (/home) → /mnt/home

SWAP → ativo

🧹 1️⃣ DESMONTAR TODAS AS PARTIÇÕES

Primeiro, desmonte tudo que foi montado em /mnt:

umount -R /mnt


-R desmonta recursivamente (EFI, home, etc. dentro de /mnt).

💧 2️⃣ DESATIVAR O SWAP (se estiver ativo)
swapoff -a

🧱 3️⃣ CONFIRMAR QUE TUDO FOI DESMONTADO
lsblk


👉 Agora nenhuma partição deve mostrar MOUNTPOINTS com /mnt.

Exemplo esperado:

NAME   MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda      8:0    0   238G  0 disk
├─sda1   8:1    0   512M  0 part 
├─sda2   8:2    0    40G  0 part 
├─sda3   8:3    0     8G  0 part 
└─sda4   8:4    0   189G  0 part 

🧩 4️⃣ REFAZER A MONTAGEM CORRETA

Agora sim, monte na ordem certa 👇

1. Monte a partição ROOT (/)
mount /dev/sda2 /mnt

2. Crie diretórios para EFI e HOME
mkdir -p /mnt/boot/efi
mkdir -p /mnt/home

3. Monte a partição EFI
mount /dev/sda1 /mnt/boot/efi

4. Monte a partição HOME
mount /dev/sda4 /mnt/home

5. Ative o SWAP
swapon /dev/sda3

🔍 5️⃣ CONFIRMAR TUDO

Rode novamente:

lsblk


Saída esperada:

NAME   FSTYPE SIZE MOUNTPOINTS
sda                   238G
├─sda1 vfat   512M /mnt/boot/efi
├─sda2 ext4    40G /mnt
├─sda3 swap     8G [SWAP]
└─sda4 ext4   189G /mnt/home


✅ Agora você tem tudo pronto pra continuar com a instalação (pacstrap).

🚀 6️⃣ (Próximo passo)

Depois que estiver tudo certo, prossiga com:

## intalação de pacotes basicos no mnt que é a raiz

pacstrap /mnt base base-devel dhcpcd nano linux linux-firmware

### criação de tabela - tabela sera gerada conforme eu tenho os discos
genfstab -U -p /mnt >> /mnt/etc/fstab

# definindo a hora
 - ln -sf /usr/share/zoneinfo/America/Sao_Paulo /etc/localtime
 listar
  - ls --color -l /etc/localtime
  - date
# acessar root nosso sistema
arch-chroot /mnt

  cat /etc/fstab

# configurar data e hora
ln -sf /usr/share/zoneinfo/America/Sao_Paulo /etc/localtime
verificar
ls --color -l /etc/localtime
date

# configurar idioma
nano /etc/locale.gen
  descomente o pt_BR.UTG-8
  pt_BR ISO-8859-9
  
locale-gen

# configurar linguagem
nano /etc/vconsole.conf
KEYMAP=br-abnt2

# Nomear a maquina
nano /etc/hostname || echo "nome-da-maquina" > /etc/hostname
nome-da-maquina

# definir a senha

passwd

### criar usuario
useradd -m -g users -G wheel, video, audio,kvm -s /bin/bash nome-do-usuario

passwd nome-do-usuario

## cria pacotes que podem ser utils
pacman -S dosfstools os-prober mtools network-manager-applet networkmanager

## concerde a usuario utilizar comando sudo

pacman -S sudo 
EDITOR=nano visudo 

Descomente a linha:
%wheel ALL=(ALL) ALL

### pacman
pacman -S grub efibootmgr
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=archboot --recheck
grub-mkconfig -o /boot/grub/grub.cfg
### sair 
exit
shutdown -h now

