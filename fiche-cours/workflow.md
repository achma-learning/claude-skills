# Workflow — Résumés de topos médicaux (fusion 3 sources)

> Workflow généralisé à **toutes les matières médicales** de l'externat marocain (FMPM Marrakech) : S5 → S8 et au-delà. Validé initialement sur *Sténose hypertrophique du pylore* (chirurgie pédiatrique), étendu à la pharmacologie, gastro-entérologie, anapath, MPR, traumato, génétique, etc.

---

## 1. Objectif

Produire, pour chaque sujet d'examen, un fichier `.md` :

- **prêt à l'emploi** dans le vault Obsidian ;
- **structuré** selon le squelette [`+++topo-plan.md`](./+++topo-plan.md) ;
- **exploitable pour la révision active** (QCM, cas clinique, mnémotechniques, logigramme) ;
- **traçable** : chaque donnée factuelle renvoie à sa source (URL + PDF) en fin de fiche.

La fiche doit tenir lieu à la fois de *cours synthétique*, de *fiche de révision J‑7* et de *banque de questions*.

---

## 2. Sources (par ordre de priorité)

| Priorité | Source | Rôle |
|---------:|--------|------|
| 1 | **Transcription du cours** — `Transcribe-data/S[n]-[matière]/*.txt` | Colonne vertébrale pédagogique : logique du prof, *pearls* cliniques, axes d'examen, ordre d'exposition. ⚠️ Erreurs de transcription fréquentes (*transhumination* → transillumination, *hyperostème* → hydrocèle, *scrotom* → scrotum) à corriger systématiquement. |
| 2 | **PDF du cours (actualisé)** — upload local ou lien Google Drive / OneDrive | Données corrigées, figures, chiffres, classifications, posologies, guidelines. **Écrase** la transcription en cas de divergence factuelle. **L'URL du PDF doit être conservée et reportée en bas de fiche.** |
| 3 | **Sources crédibles web** — recomedicales.fr, Collèges français (CFCOT, SFCE, GFHGNP, CERF, CEN, CEP, CEEDMM…), PNDS (HAS / Orphanet), sociétés savantes (SFP, SFR, SFD…), UpToDate si accessible, PubMed pour méta-analyses récentes | Couche d'arbitrage : confirme les recommandations actuelles quand (1) et (2) divergent ou datent. **Chaque URL utilisée est listée en fin de fiche.** |

### Règle de fusion

```
transcription    →  structure pédagogique + emphase examen
      +
PDF actualisé    →  override des données factuelles (chiffres, classifs, doses)
      +
web crédible     →  mise à jour des recommandations récentes
      =
fichier .md final + bloc "Sources" traçable en bas de fiche
```

---

## 3. Principe de traçabilité (nouveau)

Toute fiche produite **doit** se terminer par un bloc de sources clairement séparé :

```markdown
## Sources & Liens

### PDF du cours (fourni par le prof)
- [Nom du PDF — Pr. X — Année] : <URL du PDF ou chemin local>

### Transcription utilisée
- `Transcribe-data/S[n]-[matière]/[fichier].txt`

### Références web (guidelines, Collèges, PNDS)
- [Nom du référentiel] — <URL>
- [Recomedicales — sujet] — <https://...>
- [HAS / PNDS — sujet] — <https://has-sante.fr/...>
- …

### Pour aller plus loin
- [Article / chapitre de livre / vidéo] — <URL>
```

Règles :
1. **Toujours** reporter l'URL du PDF du cours si l'utilisateur l'a fournie (lien Drive, OneDrive, ou chemin repo).
2. **Toujours** reporter le chemin relatif de la transcription utilisée.
3. **Citer chaque URL web** effectivement consultée ; pas d'URL décorative non vérifiée.
4. Si une reco est obtenue par cross-check, indiquer en ligne `([source](url))` à la fin du paragraphe concerné, en plus du bloc récapitulatif.

---

## 4. Workflow par session

### Phase 1 — Un seul topo par session
Pas de batch. Chaque sujet mérite une synthèse focalisée. Commencer par les topos à **haut rendement examen** ou ceux dont la transcription est la plus complète.

### Phase 2 — Fournir les sources à Claude, dans cet ordre
1. **Transcription** : chemin (`Transcribe-data/...`) ou upload.
2. **PDF du cours** : upload direct **ou** lien Drive/OneDrive — *conservé pour le bloc Sources*.
3. **Points de cross-check web** : ex. *« dernière reco SFCE sur tumeurs osseuses »*, *« posologies pédiatriques amoxicilline selon AFSSAPS 2024 »*, ou *« toutes les reco récentes »*.
4. **Destination** dans le vault (cf. §6) — facultatif si évident.

### Phase 3 — Synthèse par Claude
- Extraction depuis la transcription avec correction des coquilles évidentes.
- Intégration / override via le PDF (figures, chiffres, classifs).
- Cross-reference web sur les points signalés.
- Remplissage **intégral** des sections du [`+++topo-plan.md`](./+++topo-plan.md).
- Ajout systématique : **QCM (5–10)**, **cas clinique intégratif**, **moyens mnémotechniques**, **logigramme CAT**.
- **Bloc `Sources & Liens` final** (cf. §3) avec URL du PDF + URLs web + chemin transcription.

### Phase 4 — Revue et itération
L'utilisateur relit, signale manques / erreurs (y compris sources manquantes ou URLs mortes), Claude corrige. Itérer jusqu'à validation. Passage du statut `brouillon` → `révisé` → `maîtrisé` dans le frontmatter.

---

## 5. Lancer une session

Copier le contenu de [`prompt-resume.txt`](./prompt-resume.txt), remplir les placeholders :

- `[TOPIC]` — intitulé exact du sujet
- `[TRANSCRIPTION_PATH]` — chemin de la transcription
- `[PDF]` — upload OU lien Drive OU chemin repo (⚠️ sera reporté en bas de fiche)
- `[POINTS_À_VÉRIFIER]` — points de cross-check web (ou *« toutes les reco récentes »*)
- `[DESTINATION]` — dossier cible (ex. `s7/ped-chir/`)

Exemple rapide :

> « Claude, fais-moi le résumé de **[Pathologie du CPV]**. Transcription : `Transcribe-data/S7-ped-chir/kamili-cpv.txt`. PDF du cours : https://drive.google.com/… . Cross-check web sur les reco récentes de cure de CPV. Destination : `s7/ped-chir/`. »

---

## 6. Destination du fichier produit

Organisation par semestre / matière :

| Semestre | Matière | Dossier |
|---------|---------|---------|
| S5 | Pharmacologie | `s5/pharma/` |
| S6 | Gastro-entérologie | `s6/gastro/` |
| S7 | Chirurgie pédiatrique | `s7/ped-chir/` |
| S7 | Pédiatrie médicale | `s7/ped-med/` *(si présent)* |
| S8 | Anatomie pathologique | `s8/anapath/` |
| S8 | MPR | `s8/mpr/` |
| S8 | Traumatologie | `s8/traumato/` |
| S8 | Génétique | `s8/genetique/` |
| Autre | *(à créer)* | `s[n]/[matière]/` |

Nom de fichier : `kebab-case-du-sujet.md` (ex. `invagination-intestinale-aigue.md`, `sténose-hypertrophique-pylore.md`).

---

## 7. Suivi

La liste des topos à traiter est suivie dans les **GitHub issues** du dépôt `achma-learning/resume.md` sous forme de checklist :
- une issue par module (ex. *pediatric surgery S7*, *pharmacologie S5*) ;
- cocher la case dès que la fiche est **validée** (pas seulement produite) ;
- ajouter le lien du fichier `.md` produit dans la case cochée.

---

## 8. Checklist qualité (avant de marquer un topo ✅)

- [ ] Toutes les sections de `+++topo-plan.md` sont remplies (ou marquées `⚠️ À compléter — donnée absente des sources`)
- [ ] Corrections de transcription effectuées et listées en fin de fiche
- [ ] QCM (5–10 questions) avec corrigé commenté
- [ ] Cas clinique intégratif avec corrigé
- [ ] Moyens mnémotechniques
- [ ] Logigramme CAT (mermaid ou excalidraw)
- [ ] **Bloc `Sources & Liens` final présent** avec :
  - [ ] URL du PDF du cours (si fourni)
  - [ ] Chemin de la transcription
  - [ ] URLs web consultées (Collèges, PNDS, recomedicales…)
- [ ] Frontmatter Obsidian correct (`module`, `matiere`, `semestre`, `source_pdf`, `tags`)
- [ ] Fichier déposé dans le bon dossier `s[n]/[matière]/`

---

*Workflow v2 — mise à jour {{date}} — généralisé à toutes les matières médicales, avec traçabilité des sources (PDF + web) obligatoire en fin de fiche.*
