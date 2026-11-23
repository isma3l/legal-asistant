# Norms in scope (MVP corpus)

Corpus for the RAG: Peruvian labor norms for workers in the general private-sector
regime. Two topics: CTS and gratificaciones. Files live in `data/raw/`.

Answers must be backed by these sources, cite the norm and article, and say so when
there is no source.

## Verification status

| Status | Meaning |
|---|---|
| `official` | Copy published in El Peruano. Text still has to be checked for later amendments. |
| `official-consolidated` | Consolidated text published by El Peruano ("Normas Legales Actualizadas", Editora Perú), with amendments marked. Still check for amendments after its update date. |
| `unverified` | Not an official copy. Usable for ingestion and retrieval tests only. |
| `verified` | Checked against SPIJ or El Peruano, including amendments in force, on the date given. |

No norm is `verified` yet.

## Norms

### CTS

| Norm | Title | Published | File | Source status | Validation |
|---|---|---|---|---|---|
| D.S. 001-97-TR | TUO of Legislative Decree 650, Ley de CTS | pending | `DS_001-97-TR - Ley de CTS (TUO).pdf` | MTPE copy, 2018 (not El Peruano) | pending |
| D.S. 004-97-TR | Regulation of the CTS law | pending | `DS_004-97-TR - Reglamento de CTS.pdf` | MTPE copy, 2018 (not El Peruano) | pending |
| Ley 32322 | Amends art. 42 of Legislative Decree 650 (CTS withdrawals) | 2025-05-09, El Peruano pp. 4-5 | `Ley_32322 — Modificación sobre disponibilidad de CTS.pdf` | official | pending |

Sources:
- D.S. 001-97-TR: https://www.gob.pe/institucion/mtpe/informes-publicaciones/221736-texto-unico-ordenado-de-la-ley-de-compensacion-por-tiempo-de-servicios
- D.S. 004-97-TR: https://www.gob.pe/institucion/mtpe/informes-publicaciones/221735-reglamento-de-la-ley-de-compensacion-por-tiempo-de-servicios/
- Ley 32322: https://busquedas.elperuano.pe/dispositivo/NL/2397811-1/
  (code 2397811-1 matches the one printed in the PDF)

gob.pe blocks automated clients, so the update date shown on the MTPE pages was
not checked.

### Gratificaciones

| Norm | Title | Published | File | Source status | Validation |
|---|---|---|---|---|---|
| Ley 27735 | Gratificaciones for Fiestas Patrias and Navidad | 2002-05-28 | `ley-27735-and-ds-005-2002-tr-elperuano-nla.pdf` (pp. 2-3) | official-consolidated | pending |
| D.S. 005-2002-TR | Regulation of Ley 27735 | 2002-07-04 | `ley-27735-and-ds-005-2002-tr-elperuano-nla.pdf` (pp. 3-4) | official-consolidated | pending |

Superseded files:
- `Ley_27735 - Gratificaciones.pdf`: original gazette page. Deleted on 2026-11-22
  (no usable text layer; replaced by the consolidated file).
- `DS_005-2002-TR_Reglamento_Gratificaciones_Transcripcion.pdf`: **unverified**
  transcription. Deleted on 2026-11-22 (same text as the official consolidated file;
  keeping both would store every article twice).

Sources:
- Consolidated file (both norms):
  https://diariooficial.elperuano.pe/Normas/obtenerDocumento?idNorma=60003
  (downloaded 2026-11-22).
- Ley 27735, original text (reference only, not in the corpus):
  https://leyes.congreso.gob.pe/Documentos/Leyes/27735.pdf

### Both topics

| Norm | Title | Published | File | Source status | Validation |
|---|---|---|---|---|---|
| Ley 30334 | Measures to boost the economy in 2015 | 2015-06-24, El Peruano pp. 555729-555730 | `Ley 30334 — Disposiciones sobre gratificaciones y CTS.pdf` | official | pending |

Source: https://busquedas.elperuano.pe/dispositivo/NL/1254952-1
(code 1254952-1 matches the one printed in the PDF)

## Reference material (not in the corpus)

Files in `data/reference/` help check the corpus. They are never ingested and never
cited as a source.

| File | What it is | Date |
|---|---|---|
| `6757898-guia_compensacion_tiempo_servicios.pdf` | MTPE guide on CTS (explains the law; cites TUO and regulation articles) | 2025-05-12 |

Source: https://www.gob.pe/institucion/mtpe/informes-publicaciones/6757898-guia-compensacion-por-tiempo-de-servicios

What the guide shows (checked 2026-11-22):
- It does **not** mention Ley 32322, so it was written before that law took effect.
- It mentions Ley 32027 and D.S. 003-2024-TR (free disposal of 100% of CTS because
  of the economic crisis, 2024). Neither is in the corpus.
- It cites D.S. 004-97-TR articles 19-21, which are missing from our 2018 PDF.

## Notes per norm

### D.S. 001-97-TR (TUO, Ley de CTS)
- Origin: MTPE publication on gob.pe (link above). PDF metadata: created
  2018-10-17 (Word export). Checked by hand on 2026-11-22: the MTPE page has no
  newer version.
- It cannot include the 2025 amendment by Ley 32322, nor any other change after
  2018.
- **Known limitation (accepted for the first version):** the corpus uses this 2018
  text. Rules:
  - Questions about CTS withdrawals must cite Ley 32322 as an expected article, not
    only the TUO.
  - Expected answers about any TUO article must be checked for amendments after
    2018 before they are marked as validated.
  - Replace this file if an up-to-date official consolidated text is found (for
    example, El Peruano "Normas Legales Actualizadas").
- Ley 32322 amends "article 42 of Legislative Decree 650". The TUO renumbered that
  decree. The new text starts with the same sentence as **TUO art. 41** ("El
  trabajador podrá efectuar retiros parciales de libre disposición ... 50%") and adds
  the 100% withdrawal for terminal illness or cancer. So it most likely corresponds
  to TUO art. 41 (inferred from the wording; confirm on SPIJ). TUO art. 42 (overall
  50% limit) is related.
- Withdrawal questions should list TUO art. 41 and Ley 32322 as expected articles.

### D.S. 004-97-TR (CTS regulation)
- Origin: MTPE publication on gob.pe (link above). PDF metadata: created
  2018-10-17 (Word export). Checked by hand on 2026-11-22: the MTPE page has no
  newer version. Any change after 2018 is missing (same limitation as the TUO).
- **Confirmed gap:** the MTPE guide (see "Reference material") cites articles 19, 20
  and 21 of this regulation for the procedure where the labor inspection authority
  issues the termination certificate (15 working days, positive administrative
  silence, citing the 2019 TUO of Ley 27444). Our 2018 PDF has no such articles.
  Questions on that procedure have no source in the corpus.

### Ley 32322
- The El Peruano pages also contain municipal ordinances and Leyes 32323 and 32324.
  Ingestion must isolate Ley 32322 before splitting by article.
- The sole transitory provision allows free disposal of 100% of CTS deposits only
  **until 2026-12-31**. Questions about it depend on the date; record valid-to.

### Ley 27735 and D.S. 005-2002-TR (consolidated file)
- One PDF holds both norms, so "Artículo 1" appears twice. Ingestion must split by
  norm before splitting by article.
- Page 1 is a cover and page 2 has an advertising banner. Ingestion must skip them.
- The cover says "Actualizada 05/12/2002", but the text also includes article 8-A of
  Ley 27735 (added by Ley 29351, 2009) and a note about Ley 30334 (2015). Amendments
  after 2015 still have to be checked (SPIJ).
- Amendments marked in the text:
  - Ley 27735, art. 8-A (gratificaciones not subject to contributions): added by
    Ley 29351 (2009-05-01), permanent since Ley 30334 (2015-06-24).
  - D.S. 005-2002-TR, item 3.3: corrected by erratum of 2002-07-05.
  - D.S. 005-2002-TR, item 3.4: amended by D.S. 017-2002-TR (2002-12-05).
- Ley 27735 art. 8-A and Ley 30334 art. 1 have the same text. Retrieval may return
  either; decide which one an expected answer cites.

### Ley 30334
- In scope: articles 1, 3, 4 and 5. Out of scope: article 2 (public sector) and
  article 6 (fuel price band).
- Article 5 (temporary CTS availability) must be checked for validity against later
  norms such as Ley 32322.
- The El Peruano pages also contain other norms (INIA resolutions, municipal
  ordinances). Ingestion must isolate Ley 30334.

## Pending

- [x] Source link for each norm.
- [ ] Verification date for each norm (after checking amendments in force).
- [ ] Publication date of D.S. 001-97-TR and D.S. 004-97-TR.
- [x] Readable copy of Ley 27735 (consolidated El Peruano file).
- [x] Official text of D.S. 005-2002-TR (consolidated El Peruano file).
- [ ] Check Ley 27735 and D.S. 005-2002-TR for amendments after 2015 (SPIJ).
- [x] Delete the old Ley 27735 gazette page.
- [x] Delete the D.S. 005-2002-TR transcription.
- [~] Map "article 42 of Legislative Decree 650" to its TUO article number (most likely art. 41; confirm on SPIJ).
- [ ] Check whether the TUO and its regulation PDFs include later amendments.
