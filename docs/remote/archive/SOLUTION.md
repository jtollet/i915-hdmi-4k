# Solution 4K@60Hz HDMI sur Intel Alder Lake-N

## Problème
Le moniteur Cisco Desk Pro ne pouvait pas décoder le signal 4K@60Hz via HDMI avec le driver i915, alors que le BIOS fonctionnait correctement.

## Cause racine
Le registre `PIPE_DDI_FUNC_CTL_A` différait entre le BIOS et i915 :
- **BIOS** : `0x88030111` (bit 8 activé)
- **i915** : `0x88030011` (bit 8 désactivé)

Le bit 8 (`TRANS_DDI_DP_VC_PAYLOAD_ALLOC`) est normalement utilisé pour DisplayPort, mais certains moniteurs HDMI 2.0 en ont besoin pour les modes haute bande passante (≥ 594 MHz).

## Solution
Patch dans `drivers/gpu/drm/i915/display/intel_ddi.c` (ligne ~582) :

```c
/* For 4K@60Hz, set bit 8 like BIOS does */
if (crtc_state->port_clock >= 594000)
    temp |= TRANS_DDI_DP_VC_PAYLOAD_ALLOC;
```

Cette condition active le bit 8 uniquement pour les horloges HDMI ≥ 594 MHz (4K@60Hz et résolutions supérieures).

## Vérifications effectuées

✅ **VBT Configuration** : 594 MHz max (correct)  
✅ **Scrambling HDMI 2.0** : Activé correctement (ratio 1/40)  
✅ **BPC** : 8 bits par couleur (identique au BIOS)  
✅ **Timings vidéo** : CVT standard 3840x2160@60Hz  
✅ **Registre PIPE_DDI_FUNC_CTL_A** : Maintenant identique au BIOS

## Installation

```bash
cd /home/<user>/devel/i915/linux-src
git apply /home/<user>/devel/i915/fix-4k60hz-hdmi.patch
make -j$(nproc) M=drivers/gpu/drm/i915
cd /home/<user>/devel/i915
sudo ./install_i915.sh
sudo reboot
```

## Matériel testé
- **GPU** : Intel Alder Lake-N (N100)
- **Moniteur** : Cisco Desk Pro
- **Résolution** : 3840x2160@60Hz via HDMI
- **Kernel** : 6.18.1

## Date de résolution
29 décembre 2025
