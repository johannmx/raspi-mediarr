# Plex sobre Docker en Raspberry-Pi

Con este repo podes crear tu propio server que descarga tus series y películas automáticamente, y cuando finaliza, las copia al directorio `media/` donde Plex las encuentra y las agrega a tu biblioteca.

## Todo esto con los siguientes programas que sirven para tener todo automáticamente:

- bazarr (búsqueda de subtítulos)
- lidarr (búsqueda de música)
- radarr (búsqueda de películas)
- sonarr (búsqueda de series)
- jacket (indexador de sitios torrents)
- transmission (cliente para bajar torrents)

## Requerimientos iniciales

Agregar tu usuario (cambiar `kbs` con tu nombre de usuario)

```
sudo useradd kbs -G sudo
```

Agregar esto al sudoers para correr sudo sin password

```
%sudo   ALL=(ALL:ALL) NOPASSWD:ALL
```

Agregar esta linea a `sshd_config` para que sólo tu usuario pueda hacer ssh

```
echo "AllowUsers kbs" | sudo tee -a /etc/ssh/sshd_config
sudo systemctl enable ssh && sudo systemctl start ssh
```

Instalar paquetes básicos

```
sudo apt-get update && sudo apt-get install -y \
     apt-transport-https \
     ca-certificates \
     curl \
     gnupg2 \
     software-properties-common \
     vim \
     fail2ban \
     ntfs-3g
```

Instalar Docker

```
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
echo "deb [arch=armhf] https://download.docker.com/linux/debian \
     $(lsb_release -cs) stable" | \
    sudo tee /etc/apt/sources.list.d/docker.list
sudo apt-get update && sudo apt-get install -y --no-install-recommends docker-ce docker-compose
```

Modificá tu docker config para que guarde los temps en el disco:

```
sudo vim /etc/default/docker
# Agregar esta linea al final con la ruta de tu disco externo montado
export DOCKER_TMPDIR="/mnt/storage/docker-tmp"
```

Agregar tu usuario al grupo docker 

```
# Add kbs to docker group
sudo usermod -a -G docker kbs
#(logout and login)
docker-compose up -d
```

Montar el disco (es necesario ntfs-3g si es que tenes tu disco en NTFS)
NOTA: en este [link](https://youtu.be/OYAnrmbpHeQ?t=5543) pueden ver la explicación en vivo

```
# usamos la terminal como root porque vamos a ejecutar algunos comandos que necesitan ese modo de ejecución
sudo su
# buscamos el disco que querramos montar (por ejemplo la partición sdb1 del disco sdb)
fdisk -l
# pueden usar el siguiente comando para obtener el UUID
ls -l /dev/disk/by-uuid/
# y simplemente montamos el disco en el archivo /etc/fstab (pueden hacerlo por el editor que les guste o por consola)
echo UUID="{nombre del disco o UUID que es único por cada disco}" {directorio donde queremos montarlo} (por ejemplo /mnt/storage) ntfs-3g defaults,auto 0 0 | \
     sudo tee /etc/fstab
# por último para que lea el archivo fstab
mount -a (o reiniciar)
```

## Cómo correrlo

Simplemente bajate este repo y modificá las rutas de tus archivos en el archivo (oculto) .env, y después corré:

`docker-compose up -d`

## IMPORTANTE

Para PLEX tienes que configurar los clientes para que no hagan "transcode" de los videos porque sino la RaspberryPi no aguantará la carga, un screenshot de Android acá:

<img src="https://i.imgur.com/F3kZ9Vh.png" alt="plex" width="400"/>