# Guide de soumission du patch au kernel Linux

## Patch généré
Le patch est prêt : `/home/<user>/devel/i915/0001-drm-i915-fix-4k60hz-hdmi.patch`

## Destinataires identifiés

### Mainteneurs principaux (TO:)
- Jani Nikula <jani.nikula@linux.intel.com>
- Rodrigo Vivi <rodrigo.vivi@intel.com>
- Joonas Lahtinen <joonas.lahtinen@linux.intel.com>
- Tvrtko Ursulin <tursulin@ursulin.net>

### Listes de diffusion (CC:)
- intel-gfx@lists.freedesktop.org
- intel-xe@lists.freedesktop.org
- dri-devel@lists.freedesktop.org
- linux-kernel@vger.kernel.org

## Méthode 1 : git send-email (recommandée)

### Configuration initiale
```bash
git config --global sendemail.smtpserver smtp.gmail.com
git config --global sendemail.smtpserverport 587
git config --global sendemail.smtpencryption tls
git config --global sendemail.smtpuser jerome.tollet@gmail.com
```

### Envoi du patch
```bash
cd /home/<user>/devel/i915/linux-src
git send-email --to=jani.nikula@linux.intel.com \
  --to=rodrigo.vivi@intel.com \
  --to=joonas.lahtinen@linux.intel.com \
  --to=tursulin@ursulin.net \
  --cc=intel-gfx@lists.freedesktop.org \
  --cc=intel-xe@lists.freedesktop.org \
  --cc=dri-devel@lists.freedesktop.org \
  --cc=linux-kernel@vger.kernel.org \
  /home/<user>/devel/i915/0001-drm-i915-fix-4k60hz-hdmi.patch
```

## Méthode 2 : Email manuel

### Client email
1. Ouvre ton client email
2. Crée un nouvel email en **format texte brut** (pas HTML !)
3. Sujet : **[PATCH] drm/i915: Enable TRANS_DDI_DP_VC_PAYLOAD_ALLOC for HDMI 2.0 high bandwidth modes**

### Destinataires
**TO:**
- jani.nikula@linux.intel.com
- rodrigo.vivi@intel.com
- joonas.lahtinen@linux.intel.com
- tursulin@ursulin.net

**CC:**
- intel-gfx@lists.freedesktop.org
- intel-xe@lists.freedesktop.org
- dri-devel@lists.freedesktop.org
- linux-kernel@vger.kernel.org

### Corps du message
Copie-colle le contenu complet de `0001-drm-i915-fix-4k60hz-hdmi.patch`

**IMPORTANT :** N'ajoute PAS le patch en pièce jointe, colle-le directement dans le corps de l'email.

## Méthode 3 : Via l'interface web Intel (alternative)

Intel dispose également d'un système GitLab pour les contributions :
- https://gitlab.freedesktop.org/drm/intel

Tu peux créer un Merge Request sur cette plateforme.

## Après la soumission

### Ce qui va se passer :
1. **Réponse automatique** : Tu recevras une confirmation de patchwork
2. **Revue par CI** : Les tests automatiques Intel vont tester le patch (~24h)
3. **Revue humaine** : Les mainteneurs vont examiner le code (1-2 semaines)
4. **Discussions possibles** : Ils peuvent demander des modifications
5. **Acceptation** : Si tout est OK, le patch sera intégré dans drm-intel-next

### Suivi du patch :
- Patchwork Intel : https://patchwork.freedesktop.org/project/intel-gfx/
- Tu pourras suivre l'état de ton patch sur cette plateforme

## Bonnes pratiques

✅ **À FAIRE :**
- Utiliser un email en texte brut (pas HTML)
- Répondre rapidement aux demandes de modifications
- Être patient (le processus peut prendre plusieurs semaines)
- Tester le patch sur la dernière version drm-tip si possible

❌ **À ÉVITER :**
- Envoyer le patch en pièce jointe
- Utiliser un email HTML formaté
- Envoyer des versions multiples sans attendre les retours
- Ignorer les commentaires des reviewers

## Préparation recommandée

Avant d'envoyer, vérifie que :
```bash
# Le patch s'applique proprement
cd /home/<user>/devel/i915/linux-src
git apply --check /home/<user>/devel/i915/0001-drm-i915-fix-4k60hz-hdmi.patch

# Pas d'erreurs de style
./scripts/checkpatch.pl /home/<user>/devel/i915/0001-drm-i915-fix-4k60hz-hdmi.patch
```
