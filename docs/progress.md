Plan version: v5 (updated 2026-11-21)
Stage 0: in progress (last session: 2026-11-22)
  - Theory notes: docs/learning/stage-0-why-evals-first.md
  - Comprehension answers: docs/learning/stage-0-answers.md (P2-P7 all done)
  - Norms: docs/norms.md
      - Corpus: 5 PDFs in data/raw/ (gratificaciones from El Peruano consolidated file;
        CTS TUO and regulation are 2018 MTPE copies, limitation accepted).
      - Reference only, never ingested: data/reference/ (MTPE CTS guide, 2025-05-12).
      - Still open: verification dates, SPIJ checks, confirm TUO art. 41 = DL 650 art. 42.
  - Questions: evals/questions.yaml (10/50, all gratificaciones, all verified_by_hand: false)
      - Format agreed: expected_articles as groups (one hit per group, alternatives
        inside a group); group (dev/held-out) drawn at random once all 50 exist.

Next session:
  1. User hand-checks the 10 questions against the official PDFs and sets
     verified_by_hand: true. Start with grat-009: is DS 005 art. 2 enough, or is
     art. 3 (numeral 3.4, 1/30 deduction) also needed?
  2. Next batch: article_number questions, trap questions (candidates: termination
     certificate procedure, DS 004 arts. 19-21 missing; Ley 32027), first CTS questions.
  3. Grow to 50, random split 35/15, then stage 0 commit + note in docs/decisions/.
