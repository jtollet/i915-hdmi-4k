# Création d'un Merge Request sur GitLab

## Prérequis

1. Compte GitLab sur https://gitlab.freedesktop.org/
2. Fork du repository drm/xe ou drm/intel

## Étapes

### 1. Fork le repository (une seule fois)

Visite : https://gitlab.freedesktop.org/drm/intel
Clique sur **Fork** en haut à droite

### 2. Ajoute ton fork comme remote

```bash
cd /home/<user>/devel/i915/linux-src
git remote add myfork git@gitlab.freedesktop.org:TON_USERNAME/intel.git
```

### 3. Pousse ta branche

```bash
# Crée une branche pour ton patch
git checkout -b fix-hdmi-4k60hz

# Pousse vers ton fork
git push myfork fix-hdmi-4k60hz
```

### 4. Crée le Merge Request

1. Va sur ton fork : https://gitlab.freedesktop.org/TON_USERNAME/intel
2. Clique sur **Create merge request**
3. Remplis :
   - **Title** : `drm/i915: Enable TRANS_DDI_DP_VC_PAYLOAD_ALLOC for HDMI 2.0 high bandwidth modes`
   - **Description** : Copie le message de commit + ajoute `Closes: #6868`
   - **Target branch** : drm-intel-next (ou main)
   - **Assignees** : Laisse vide (les mainteneurs s'assigneront)
   - **Labels** : i915, HDMI, bug

4. Dans la description, ajoute :
   ```
   Closes: #6868
   ```
   Cela fermera automatiquement l'issue quand le MR sera mergé.

### 5. Template de description MR

```markdown
## What does this MR do?

Fixes 4K@60Hz HDMI display on certain monitors (Cisco Desk Pro) with Intel Alder Lake by enabling TRANS_DDI_DP_VC_PAYLOAD_ALLOC bit for high bandwidth HDMI modes.

## Why was this MR needed?

Some HDMI 2.0 monitors require the TRANS_DDI_DP_VC_PAYLOAD_ALLOC bit (bit 8) to be set for modes >= 594 MHz, even though this bit is typically used for DisplayPort. The BIOS sets this bit, but i915 does not, causing the monitor to be unable to decode the signal.

## What are the relevant issue numbers?

Closes: #6868

## Testing

Tested on:
- Intel Alder Lake-N (N100)
- Cisco Desk Pro monitor via HDMI
- 3840x2160@60Hz working correctly after patch

Before patch: Monitor shows "unable to decode signal"
After patch: 4K@60Hz works perfectly

## Checklist

- [x] Code tested locally
- [x] Passes checkpatch.pl with 0 errors, 0 warnings
- [x] Includes Signed-off-by
- [x] References issue #6868
```
