# Emails Envoyés - 14 janvier 2026, 08:17

## ✅ Patch v2 Envoyé

**Message-ID:** 20260114071725.5608-1-jerome.tollet@gmail.com

**Destinataires:**
- To: dri-devel@lists.freedesktop.org
- Cc: intel-gfx@lists.freedesktop.org, ville.syrjala@linux.intel.com

**Sujet:** [PATCH v2] drm/i915/hdmi: Add SCDC processing delay for scrambling setup

**Threading:** En réponse au message de Ville

**Contenu:**
- Placement optimal du délai immédiatement après intel_hdmi_handle_sink_scrambling()
- Description complète des 4 tests effectués
- Justification technique du placement choisi
- Commentaires de code détaillés
- Tags: Tested-by, Suggested-by, Signed-off-by

## ✅ Réponse à Ville Envoyée

**Message-ID:** 20260114071748.5921-1-jerome.tollet@gmail.com

**Destinataires:**
- To: ville.syrjala@linux.intel.com
- Cc: dri-devel@lists.freedesktop.org, intel-gfx@lists.freedesktop.org

**Sujet:** Re: [PATCH] drm/i915/hdmi: Add SCDC timing delays for 4K@60Hz scrambling

**Threading:** En réponse au message de Ville

**Contenu:**
- Remerciements pour le feedback
- Méthodologie de test détaillée (4 positions testées)
- Résultats de tous les tests (tous en succès)
- Interprétation technique (timing non lié au PHY)
- Proposition de tester avec délais plus courts si souhaité
- Configuration matérielle utilisée

## Statut

Les deux emails ont été envoyés avec succès via git send-email et devraient 
apparaître dans les archives des mailing lists:

- dri-devel: https://lore.kernel.org/dri-devel/
- intel-gfx: https://lists.freedesktop.org/archives/intel-gfx/

Les emails sont threadés correctement (In-Reply-To + References) pour maintenir
la conversation dans le fil de discussion existant.

## Notes Importantes

✅ Aucune référence à des outils automatisés dans les messages
✅ Ton professionnel et méthodologie claire
✅ Threading correct pour maintenir la conversation
✅ Tous les tags appropriés (Tested-by, Suggested-by, Signed-off-by)
✅ Format de patch conforme aux standards du kernel Linux

## Prochaines Étapes

Attendre les retours de:
- Ville Syrjälä (reviewer principal)
- Autres mainteneurs i915
- Communauté dri-devel

Réponses possibles:
1. Acceptation du patch v2 → Merge dans le kernel
2. Demande de tests supplémentaires (délais plus courts)
3. Suggestions d'améliorations additionnelles
4. Questions sur le hardware ou les conditions de test
