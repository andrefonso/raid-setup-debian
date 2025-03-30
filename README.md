# Configuração de RAID0, RAID1 e RAID5 no Debian/Ubuntu

Este tutorial ensina como instalar e configurar RAID 0, RAID 1 e RAID 5 em distribuições baseadas no Debian/Ubuntu. Também abordamos como recuperar erros e desfazer a configuração de RAID.
---

## Pré-requisitos

- Mínimo de 2 discos para RAID 0 ou RAID 1
- Mínimo de 3 discos para RAID 5
- Acesso root ou sudo
- Pacote `mdadm` instalado

### Instalando o mdadm
```bash
sudo apt update && sudo apt install mdadm -y
```

---
## Criando RAID

### RAID 0 (Striping - desempenho)
```bash
sudo mdadm --create --verbose /dev/md0 --level=0 --raid-devices=2 /dev/sdX /dev/sdY
```
Substitua `/dev/sdX` e `/dev/sdY` pelos discos reais.

### RAID 1 (Espelhamento - redundância)
```bash
sudo mdadm --create --verbose /dev/md0 --level=1 --raid-devices=2 /dev/sdX /dev/sdY
```

### RAID 5 (Desempenho e segurança)
```bash
sudo mdadm --create --verbose /dev/md0 --level=5 --raid-devices=3 /dev/sdX /dev/sdY /dev/sdZ
```

### Verificando o RAID
```bash
cat /proc/mdstat
sudo mdadm --detail /dev/md0
```

### Criando sistema de arquivos e montando RAID
```bash
sudo mkfs.ext4 /dev/md0
sudo mkdir /mnt/raid
sudo mount /dev/md0 /mnt/raid
```

Para montagem persistente, adicione ao `/etc/fstab`:
```bash
echo '/dev/md0 /mnt/raid ext4 defaults,nofail,discard 0 0' | sudo tee -a /etc/fstab
```

---
## Recuperação de RAID

### Simulando falha de um disco (RAID 1 e RAID 5)
```bash
sudo mdadm --fail /dev/md0 /dev/sdX
sudo mdadm --remove /dev/md0 /dev/sdX
```

### Substituindo disco com falha
```bash
sudo mdadm --add /dev/md0 /dev/sdX
```

---
## Removendo RAID

1. Desmonte o RAID:
```bash
sudo umount /mnt/raid
```
2. Pare o array RAID:
```bash
sudo mdadm --stop /dev/md0
```
3. Remova a configuração:
```bash
sudo mdadm --zero-superblock /dev/sdX /dev/sdY
```
4. Atualize a configuração:
```bash
sudo mdadm --detail --scan > /etc/mdadm/mdadm.conf
```
