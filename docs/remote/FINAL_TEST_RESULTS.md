# Résultats Finaux des Tests - Réponse à Ville Syrjälä

## Contexte

Ville a suggéré d'identifier précisément quelle opération nécessite le délai en déplaçant 
progressivement le délai plus tôt dans la séquence de modeset jusqu'à ce que ça cesse de 
fonctionner.

## Séquence de Modeset Testée

La fonction intel_ddi_enable_hdmi() exécute ces opérations dans l'ordre:

1. intel_hdmi_handle_sink_scrambling() - Configure SCDC (ligne 3416)
2. hsw_prepare_hdmi_ddi_buffers() - Prépare les buffers HDMI (ligne 3423)
3. mtl_ddi_enable_d2d() - Active D2D pour C10/C20 Phy (ligne 3427)
4. encoder->set_signal_levels() - Configure les niveaux de signal (ligne 3429)
5. Display WA #1143 - Workaround display pour SKL/KBL/CFL (ligne 3432)
6. intel_ddi_power_up_lanes() - Active les lanes (ligne 3466)
7. intel_ddi_buf_enable() - Active le buffer DDI (ligne 3504)
8. intel_hdmi_poll_for_scrambling_enable() - Polling du statut (ligne 3506)

## Résultats des Tests

| Test | Position du Délai (150ms) | Ligne | Résultat |
|------|---------------------------|-------|----------|
| Test 1 | Avant intel_ddi_buf_enable() | ~3504 | ✅ SUCCÈS |
| Test 2 | Avant intel_ddi_power_up_lanes() | ~3466 | ✅ SUCCÈS |
| Test 3 | Après set_signal_levels() | ~3429 | ✅ SUCCÈS |
| Test 4 | Après hsw_prepare_hdmi_ddi_buffers() | ~3424 | ✅ SUCCÈS |

## Conclusion

**TOUS les tests ont réussi!**

Le délai de 150ms fonctionne peu importe où il est placé dans la séquence de modeset, 
du point le plus précoce (après buffer prep) au point le plus tardif (avant buf_enable).

### Implications

Cette découverte est extrêmement révélatrice:

1. **Le timing n'est PAS lié à une opération PHY/DDI spécifique**
   - Si c'était le cas, certains tests auraient échoué
   - Le délai fonctionne avant OU après n'importe quelle opération PHY

2. **Le monitor a besoin de temps pour traiter la config SCDC**
   - La configuration SCDC se fait en premier via intel_hdmi_handle_sink_scrambling()
   - Le monitor a besoin de ~150ms pour traiter cette configuration
   - Ce délai peut être placé N'IMPORTE OÙ après la config SCDC
   - Tant qu'il y a 150ms quelque part dans la séquence, ça fonctionne

3. **Comportement conforme à HDMI 2.0 spec (section 10.4.1.7)**
   - La spec dit que le sink peut désactiver le scrambling s'il ne détecte pas 
     de signal scramblé dans les 100ms
   - Notre monitor (Cisco Desk Pro) semble avoir besoin de plus de temps (~150ms)
     pour PRÉPARER sa détection, pas pour détecter le signal lui-même

## Recommandation pour le Patch Final

Basé sur ces résultats, je recommande de placer le délai **AU PLUS TÔT** dans la 
séquence, c'est-à-dire **immédiatement après intel_hdmi_handle_sink_scrambling()**.

### Justification

1. **Clarté du code**: Place le délai juste après l'opération qui le nécessite (config SCDC)
2. **Performance**: Permet aux opérations PHY de s'exécuter pendant que le monitor traite
3. **Maintenabilité**: Évident pour les futurs développeurs pourquoi le délai est là
4. **Documentation**: Commentaire explicite liant le délai à la config SCDC

## Prochaines Étapes

1. Créer un patch propre avec le délai placé après intel_hdmi_handle_sink_scrambling()
2. Tester avec des valeurs plus petites (100ms, 125ms) pour trouver le minimum requis
3. Répondre à Ville sur la mailing list avec ces résultats
4. Soumettre un v2 du patch avec le placement optimal

## Configuration Matérielle des Tests

- **CPU**: Intel Alder Lake-N N100 (Gen 12.0)
- **Monitor**: Cisco Desk Pro (HDMI 2.0)
- **Résolution**: 3840x2160@60Hz (594 MHz, scrambling requis)
- **Kernel**: 6.18.1-061801-generic
- **Driver**: i915 (version modifiée avec patches de test)

## Date des Tests

14 janvier 2026

## Testé par

Jerome Tollet <jerome.tollet@gmail.com>
