# NVIDIA Legacy 390xx + Bumblebee en MX Linux (Trixie / Kernel 6.12)

Configuracion funcional de Optimus (Intel HD 4400 + NVIDIA GeForce GT 720M)
en un portatil con MX Linux, kernel 6.12.101+deb13-amd64, driver
nvidia-legacy-390xx-driver 390.157-13~mx25.

## Hardware
- GPU discreta: NVIDIA GF117M GeForce GT 720M (Fermi)
- GPU integrada: Intel HD Graphics 4400 (Haswell)
- Bus de la NVIDIA: PCI:4:0:0 (0000:04:00.0)

## Estado final verificado
Sin optirun  ->  Mesa Intel(R) HD Graphics 4400 (HSW GT2)
Con optirun  ->  GeForce GT 720M/PCIe/SSE2

## Instalacion del driver
sudo apt install nvidia-legacy-390xx-driver nvidia-legacy-390xx-kernel-dkms

El paquete de MX Linux (~mx25) ya trae los backports necesarios para
compilar contra kernel 6.12. No hizo falta parchear nada a mano.

## Pasos de configuracion aplicados

1. Bloquear nouveau en /etc/modprobe.d/blacklist-nouveau.conf:
blacklist nouveau
options nouveau modeset=0
Luego: sudo update-initramfs -u

2. Instalar Bumblebee:
sudo apt install bumblebee primus bbswitch-dkms
sudo usermod -aG bumblebee $USER

3. En /etc/bumblebee/bumblebee.conf:
Driver=nvidia
Bridge=primus
PMMethod=bbswitch
KernelDriver=nvidia_legacy_390xx (dentro de [driver-nvidia])

4. En /etc/bumblebee/xorg.conf.nvidia, dentro de la seccion Device:
BusID       "PCI:4:0:0"

5. Causa raiz de que GLX fallara en el escritorio normal:
El paquete nvidia cambia el proveedor global de GLX a NVIDIA.
Arreglo:
sudo update-alternatives --set glx /usr/lib/mesa-diverted
sudo reboot

## Verificacion
dkms status
glxinfo | grep "OpenGL renderer"
optirun glxinfo | grep "OpenGL renderer"
cat /proc/acpi/bbswitch

## Uso en Steam
Opciones de lanzamiento del juego:
optirun %command%

## Congelar versiones para no romper esto con una actualizacion
sudo apt-mark hold nvidia-legacy-390xx-driver nvidia-legacy-390xx-kernel-dkms nvidia-legacy-390xx-kernel-support nvidia-legacy-390xx-driver-libs bbswitch-dkms

## Restaurar configuracion desde este repo si algo la sobreescribe
sudo cp config/bumblebee.conf /etc/bumblebee/bumblebee.conf
sudo cp config/xorg.conf.nvidia /etc/bumblebee/xorg.conf.nvidia
sudo cp config/blacklist-nouveau.conf /etc/modprobe.d/blacklist-nouveau.conf
sudo update-initramfs -u
sudo service bumblebeed restart
