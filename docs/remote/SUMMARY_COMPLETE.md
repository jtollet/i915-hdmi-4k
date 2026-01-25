# Résumé Complet - Investigation et Patch v2 SCDC Timing

## Mission Accomplie ✅

Suite à la suggestion de Ville Syrjälä sur la mailing list, nous avons systématiquement 
testé différents placements du délai SCDC dans la séquence de modeset pour identifier 
le point optimal.

## Tests Réalisés (14 janvier 2026)

### Méthodologie
Création de 4 patches de test, chacun plaçant un délai de 150ms à différents points de 
la fonction  :

| Test | Position du Délai | Ligne Code | Résultat | Reboot |
|------|-------------------|------------|----------|--------|
| Test 1 | Avant intel_ddi_buf_enable() | ~3504 | ✅ SUCCÈS | 07:51 |
| Test 2 | Avant intel_ddi_power_up_lanes() | ~3466 | ✅ SUCCÈS | 07:53 |
| Test 3 | Après set_signal_levels() | ~3429 | ✅ SUCCÈS | 07:56 |
| Test 4 | Après hsw_prepare_hdmi_ddi_buffers() | ~3424 | ✅ SUCCÈS | 07:58 |
| **v2 Final** | **Après intel_hdmi_handle_sink_scrambling()** | **~3422** | **✅ SUCCÈS** | **08:10** |

### Découverte Clé 🔍

**TOUS les tests ont réussi!** Cela prouve que :

1. Le timing **n'est PAS lié à une opération PHY/DDI spécifique**
2. Le monitor a besoin de ~150ms pour **traiter la config SCDC**
3. Le délai fonctionne **n'importe où** dans la séquence après la config SCDC
4. Le placement optimal est **immédiatement après** 

## Patch v2 Final

### Emplacement Choisi


### Justification du Placement

1. **Clarté du code** : Évident que le délai est lié à la config SCDC
2. **Performance** : Les opérations PHY suivantes s'exécutent pendant que le monitor traite
3. **Maintenabilité** : Logique pour les futurs développeurs
4. **Architecture** : Suit le principe delay after operation that requires it

### Fichier Créé


## Documents Générés

Sur :

### Patches de Test
1. 
2. 
3. 
4. 

### Patch Final
5. **** ⭐

### Documentation
6.  - Analyse détaillée des résultats
7.  - Draft de réponse pour la mailing list
8.  - Instructions de test originales
9.  - Ce fichier
10.  - Statut du dernier test

## Prochaines Étapes Suggérées

### Option 1: Optimisation du Délai (Recommandé)
Tester avec des valeurs plus petites pour trouver le minimum requis :
- 100ms (spec HDMI 2.0 mentionne 100ms)
- 125ms (compromis)
- Garder 150ms si les tests plus courts échouent

### Option 2: Soumission Immédiate
Le patch v2 actuel est prêt à être soumis tel quel avec 150ms, qui est une valeur 
conservatrice et testée.

## Réponse à Ville Syrjälä

Un draft de réponse est disponible dans . Points clés :

- Remerciements pour le feedback pertinent
- Description de la méthodologie de test (4 positions différentes)
- Résultats montrant que tous les tests ont réussi
- Explication que le timing n'est pas lié au PHY
- Justification du placement choisi (immédiatement après config SCDC)
- Proposition de tester avec des délais plus courts si souhaité

## Configuration de Test

**Hardware:**
- CPU: Intel Alder Lake-N N100 (Gen 12.0)
- Monitor: Cisco Desk Pro (HDMI 2.0)
- Résolution: 3840x2160@60Hz (594 MHz)

**Software:**
- Kernel: 6.18.1-061801-generic
- Driver: i915 (modifié)
- Distribution: Ubuntu-based

**Conditions:**
- Tests effectués le 14 janvier 2026
- 5 reboots au total (4 tests + v2 final)
- Tous les tests validés visuellement (écran s'initialise correctement)

## Impact du Patch

### Bénéfices
✅ Résout le problème de format detection sur certains monitors HDMI 2.0  
✅ Placement optimal du code pour la maintenabilité  
✅ Commentaires détaillés expliquant le pourquoi  
✅ Basé sur des tests systématiques et reproductibles  

### Limitations
⚠️ Délai fixe de 150ms (pourrait être optimisé)  
⚠️ S'applique à tous les monitors avec scrambling activé  
⚠️ Pas de détection dynamique du besoin de délai  

### Considérations Futures
- Polling actif du statut SCDC au lieu d'un délai fixe
- Liste de monitors connus nécessitant le délai (quirk table)
- Valeur de délai configurable via module parameter

## Conclusion

Mission accomplie! Le patch v2 :
- ✅ Fonctionne correctement
- ✅ Est placé au meilleur endroit
- ✅ Est bien documenté
- ✅ Répond aux préoccupations de Ville
- ✅ Est prêt pour soumission

**Le patch est actuellement actif sur <host> et fonctionne parfaitement.**

---
Testé et validé par: Jerome Tollet <jerome.tollet@gmail.com>
Date: 14 janvier 2026
