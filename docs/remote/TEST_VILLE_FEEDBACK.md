# Tests pour Réponse à Ville Syrjälä

## Objectif
Identifier exactement où placer le délai dans la séquence de modeset HDMI.

## Patches Créés

1. **test1_delay_before_buf_enable.patch** - Délai avant activation du buffer DDI
2. **test2_delay_before_power_up_lanes.patch** - Délai avant power-up des lanes  
3. **test3_delay_after_signal_levels.patch** - Délai après config signal levels
4. **test4_delay_after_buffer_prep.patch** - Délai après préparation buffers

## Comment Tester

Pour chaque patch:

```bash
cd ~/devel/i915/linux-src
git reset --hard HEAD
git apply ../test1_delay_before_buf_enable.patch  # Remplacer par test2, test3, test4
cd ..
./install_i915.sh
# Redémarrer
xrandr --output HDMI-1 --mode 3840x2160 --rate 60
dmesg | grep -i test
```

## Ordre Recommandé

Commencer par **Test 1**, puis progresser vers Test 4 selon les résultats.

## Résultats à Noter

- Est-ce que 4K@60Hz fonctionne ?
- Y a-t-il des messages d'erreur dans dmesg ?
- L'affichage est-il stable après reboot/DPMS ?

Ces informations permettront de répondre précisément à Ville sur le placement optimal du délai.
