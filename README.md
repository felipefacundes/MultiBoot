##### Criando um Pendrive Multiboot no ArchLinux, com Ferramentas administrativas, diversas distrinuições Linux e com Windows.
</br>
</br>

###### Iremos usar tanto o sistema UEFI para BOOT como MBR (para sistemas mais antigos).

###### Primeiro formate o pendrive com uma partição pequena de `40M` como `FAT32`, para usar como `EFI`. A tabela de partição não precisa ser em GPT, pelo contrári, é recomendado que seja tabela `DOS`, pois a tabela GPT tem limitações, a primeira é a escrita em `MBR`, o GRUB não escreve em MBR se a tabela for GPT. A segunda limitação é que esta tabela somente aceita partições de até 2T, limitação que não existe com a tabela `DOS`.

##### Instale algumas Ferramentas no seu sistema, antes de gerir o pendrive:
`sudo pacman -S arch-install-scripts exfatprogs mtools dosfstools libisoburn`

##### Detectando qual unidade é o pendrive:
`sudo fdisk -l`

##### Criando a tabela de partição no pendrive:
`sudo cfdisk -z /dev/sdX` # Onde X é a letra que corresponde ao seu pendrive.

###### Ao escolher a tabela DOS crie a primeira partição de `40M` a segunda com o tamanho inteiro, e se quiser ter o Instalador do Windows no pendrive basta criar uma terceira partição como `FAT32`, porém está partição tem que ser no tamanho exato da imagem ISO, exemplo, para uma imagem do Windows 8.1, essa partição precisará ter 4,5G.

###### Não crie mais que três partições, pois a partir da 4ª partição o pendrive ficará lento. Então, só crie 3 partições. A primeira use como FAT32 /EFI. A segunda, será a maior partição como `EXT4` e a terceira para Windows Installer como `FAT32`.

##### Supondo que o seu pendrive seja a segunda unidade de disco com a letra `B`, lembrando, antes detecte a unidade com `sudo fdisk -l`. Então, formate assim:
```
sudo mkfs.vfat -F32 -n EFI /dev/sdb1
sudo mkfs.ext4 /dev/sdb2
sudo mkfs.vfat -F32 -n WIN_INSTALL /dev/sdb3
```
##### Montando às unidades - substitua a letra `B` pela letra correspondente do seu pendrive:
```
sudo mount /dev/sdb2 /mnt
sudo mkdir -p /mnt/boot/EFI
sudo mkdir -p /mnt/WIN_INSTALL
sudo mkdir -p /mnt/ISOs
sudo mount /dev/sdb1 /mnt/boot/EFI
sudo mount /dev/sdb3 /mnt/WIN_INSTALL
sudo chown -R "$USER":users /mnt/
```
##### Instale o básico no pendrive, NÂO instale o kernel
`sudo pacstrap -i /mnt efibootmgr exfatprogs grub pacman dosfstools libisoburn os-prober mtools fuse2 freetype2 ntfs-3g sed f2fs-tools parted fatresize tar psmisc pciutils usbutils libusb-compat memtest86`

###### Baixe às imagems ISO que deseja que o pendrive tenha no BOOT.
###### Para Windows, faça o seguinte:
```
mkdir ~/WIN
sudo modprobe loop
sudo mount -o loop Instalador-do-Windows.iso ~/WIN

cp -rf ~/WIN/* /mnt/WIN_INSTALL/ && sync

sudo umount -R ~/WIN && rm -rf ~/WIN
```

###### Se você você for ter uma imagem de alguma distribuição Linux no pendrive, copie no pendrive na pasta ISOs.

`cp -f imagem-da-distribuição-linux.iso /mnt/ISOs/`

###### Editando o `40_custom` do GRUB para ter múltiplos boots no pendrive. Use o editor de texto da sua preferência, no exemplo aqui será o `vim`

`sudo vim /mnt/etc/grub.d/40_custom`

##### Exemplo de `40_custom` com o Instalador do Windows na partição do pendrive e ArchLinux ISO, lembrando que você poderá ter qualquer distribuição Linux no pendrive, mas para cada distribuição Linux os parâmetros da ISO são diferentes:

###### Antes de tudo verifique o UUID de cada partição do pendrive com o comando: `sudo blkid` e substitua onde está `1234-5678` pelo UUID correspondente da partição onde estão às ISOs e a partição do Windows Installer.

```
menuentry "ArchLinux-2020.iso" --class dvd {
      insmod part_msdos
      insmod search_fs_uuid
      set root=UUID=1234-5678
      set isofile="/ISOs/ArchLinux-2020.iso" root=UUID=1234-5678
      set dri="free"
      search --no-floppy -f --set=root $isofile
      probe -u $root --set=abc
      set pqr="/dev/disk/by-uuid/$abc"
      loopback loop $isofile
      linux (loop)/arch/boot/x86_64/vmlinuz img_dev=$pqr img_loop=$isofile driver=$dri
      initrd (loop)/arch/boot/x86_64/archiso.img
}

menuentry "Windows 10 Installer Partition" --class windows --class os {
        insmod part_msdos
        insmod chain
        insmod exfat
        insmod fat
        insmod search_fs_uuid
        set root=UUID=1234-5678
        search --no-floppy --fs-uuid --set=root 1234-5678
        chainloader /efi/boot/bootx64.efi root=UUID=1234-5678
}
```

##### Após às devidas modificações no arquivo `40_custom` - lembrando que você deve alterar conforme o seu caso - entre no Pendrive e faça a atualização do GRUB:

##### Agora entre no mini sistema arch e vamos gerir o GRUB:
```
sudo arch-chroot /mnt

grub-install --target=i386-pc --recheck /dev/sdb   #Este comando é para instalar o GRUB na MBR

grub-install --verbose --recheck --force --target=x86_64-efi --efi-directory=/boot/EFI --bootloader-id=MULTIBOOT --removable

grub-mkconfig -o /boot/grub/grub.cfg

exit    #Ou control + D - para sair do chroot

sudo umount -R /mnt
```

##### Pendrive pronto, basta reiniciar o sistema e dar boot pelo pendrive. Não esqueça de ler esse artigo com cautela para fazer tudo certinho ;)

###### Segue alguns exemplos de `40_custom`:
[Exemplo 1](https://github.com/felipefacundes/MultiBoot/blob/main/40_custom)

[Exemplo 2](https://github.com/felipefacundes/MultiBoot/blob/main/exemplos/40_custom.exemplo.2)

[Exemplo 3](https://github.com/felipefacundes/MultiBoot/blob/main/exemplos/40_custom.exemplo.3.diversos)
