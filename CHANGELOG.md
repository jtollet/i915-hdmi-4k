# Changelog

Chronological summary of the patch variants created during the HDMI 4K@60Hz
investigation.

## 2025-12-30
- Initial timing-delay proposal (add fixed sleeps before SCDC config and after
  DDI enable) to stabilize 4K@60Hz scrambling on sensitive sinks.

## 2026-01-07
- Polling variants for TMDS_Scrambler_Status (200 ms / 300 ms) to align with
  HDMI 2.0 spec behavior.

## 2026-01-08
- v2 polling patch posted (poll for up to 200 ms after enabling HDMI), with
  formal SCDC status polling helpers.

## 2026-01-13
- Test patch series moving the delay earlier in the HDMI enable sequence to
  isolate the minimal required timing window.

## 2026-01-14
- v2 SCDC processing delay patch: move the delay immediately after SCDC
  configuration for clarity and maintainability, plus improved rationale.

## 2026-01-21
- Ankit Nautiyal suggests considering panel-specific quirk instead of generic
  solution, given that all delay placements worked in testing.

## 2026-01-28
- Follow-up message asking maintainers for preferred approach: generic polling,
  panel-specific quirk, or fixed delay.

## 2026-02-09
- Gentle ping after 10 days with summary of three possible approaches awaiting
  maintainer feedback.

## 2026-02-14
- **Isolated testing on kernel 6.18.7 confirms: SCDC polling patch alone works
  perfectly. Delay patch is placement-dependent and less robust.**
- Conclusion: Generic SCDC polling (Ankit's patch) is the recommended solution
  as it aligns with HDMI 2.0 spec and Windows behavior.

## Additional variants (archive)
- Earlier combined fixes and experimental patches kept for reference in
  `patches/local-archives` and `patches/remote-archive`.

## Notes
- Full patch list: `docs/PATCHES_INDEX.md`
- Supporting docs: `docs/DOCS_INDEX.md`
- Mail and issue links: `docs/MAILS.md`
