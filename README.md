# garcia-sanabria-plant-classifier

Image classifier that recognizes 6 plant species from **Parque García Sanabria**
(Santa Cruz de Tenerife), plus an "other" class for plants and objects outside the collection.

An educational project built as my first hands-on experience with applied deep learning,
following the approach of the [fast.ai](https://course.fast.ai/) course (Practical Deep Learning
for Coders). Built with [fastai](https://docs.fast.ai/) using transfer learning, and
deployed online as an interactive game.

> **Note:** this project is developed *iteratively*. Each model version comes from analyzing
> the errors of the previous one, gathered by **testing in the field**, not just by reading the
> error_rate. All versions are documented below, not only the one online —
> because the process of diagnosis and correction is part of the work.

**Playable demo:** https://planthuntsanabria.netlify.app/

---

## What it does

Given a photo of a plant, the model predicts which of the 7 categories it belongs to:

| Category | Scientific name |
|----------|-----------------|
| Drago | *Dracaena draco* |
| Mejorana | *Origanum majorana* |
| Olivo | *Olea europaea* |
| Palmera canaria | *Phoenix canariensis* |
| Alcalifa | *Acalypha wilkesiana* |
| Palmera cola de pescado ramificada | *Caryota* |
| Other | (plants and objects outside the collection) |

### Images per class

| Category | v1 | v2 | v3 |
|----------|----|----|----|
| Drago | 27 | 29 | 29 |
| Mejorana | 33 | 33 | 33 |
| Olivo | 27 | 27 | 27 |
| Palmera canaria | 30 | 29 | 29 |
| Alcalifa | 23 | 26 | 26 |
| Palmera cola de pescado | 27 | 32 | — |
| Other | 29 | 30 | 30 |
| **Total** | **196** | **206** | **174** |

The rebalancing in v2 is not random: the reinforced classes (palmera cola de pescado, alcalifa)
are exactly the ones the error analysis had identified as weakest. v3 is not a new dataset,
but v2 with one class removed, in order to isolate its effect.

---

## Method (shared by all versions)

- **Transfer learning** from a pretrained **ResNet34**.
- **Data augmentation** (`aug_transforms()`) to compensate for the small dataset.
- Images resized to **224×224**.
- Train/validation split: **80/20** (`valid_pct=0.2`, `seed=42`).
- Training with `fine_tune`, number of epochs chosen experimentally.

---

## Model versions

### v1 — baseline (03/08/2026)

- **Dataset:** 196 images, 7 classes (23–33 photos per class).
- **Training:** 8 epochs (chosen iteratively: 3 → 5 → 8, monitoring the `valid_loss`).
- **Result:** error_rate ≈ 0.077 (~92% on the validation set, ~39 photos).
- **Post-test diagnosis:** field testing revealed problems not visible in the error_rate.
  In particular, the model tended to "hallucinate" the `drago` class when the image had a
  **uniform background (sky)**, and struggled with **trunks** and with the **palmera cola
  de pescado**. The ~92% was also inflated by near-duplicate shots split between training
  and validation.

### v2 — diagnosis-driven correction (06/08/2026)

- **Dataset:** 206 images. Targeted changes from v1, all stemming from the error analysis:
  - removed `drago` and `palmera_canaria` photos with too much sky in the background
    (hypothesis: the model was associating the sky with the `drago` class);
  - removed some near-duplicate shots from various classes;
  - added `drago` photos with no sky or with little sky;
  - added "full view" photos to the classes that were poorly recognized when framed in full;
  - enriched the `other` class (whitish plants, objects) and rebalanced it by removing
    some excess trunks.
- **Training:** 8 epochs.
- **Result:** error_rate ≈ 0.122 (~88% on the validation set, ~41 photos).
- **This is the version currently online.**

> **Why v2 scores lower than v1 despite being better.** The v2 validation set is
> *cleaner* (fewer near-duplicate shots) and *stricter* (more varied, harder photos). A lower
> number on a more honest test does not mean a worse model: v1's ~92% was partly inflated.
> The real check on the improvements (e.g. the sky/drago problem) is done by testing in the
> field, not by reading the error_rate.

### v3 — 6-class experiment, discarded (06/08/2026)

- **Dataset:** 174 images — v2 **without** the `palmera cola de pescado ramificada` class.
- **Motivation:** that species was the most problematic one in field tests
  (5 correct recognitions out of 22 attempts). The hypothesis was that removing it would
  simplify the problem and reduce confusion among the other classes.
- **Result:** error_rate ≈ 0.206 (~79% on the validation set, ~35 photos) — the lowest
  value of the three versions.
- **What it fixed:** **generic trunks** were no longer assigned to `olivo` or
  `palmera_canaria`, but correctly rejected as "no match". This was the problem v2 had
  failed to solve.
- **What it broke:** `palmera_canaria` went from reliable to **recognized 1 time out of
  26**, under the same conditions and with the same type of shots as v2.
- **Decision:** version discarded. The gain on trunks did not make up for such a sharp
  regression on one of the main species.

> **Methodological note on comparisons.** The three accuracy values (92% / 88% / 79%) **are not
> directly comparable**: each version has a different dataset and therefore a different
> validation set, in both composition and difficulty. They are useful for tracking each
> individual version, not for ranking them against each other.

---

## Field tests

The error_rate alone proved insufficient to describe the model's real-world behavior.
For this reason each version was tested by taking live photos, and the results are
recorded in full in the decision log.

| Test | Model | Photos | What it revealed |
|------|-------|--------|------------------|
| Test 1 | v1 | 94 | `drago` was also predicted for objects (shoes, keys, bottles) on uniform surfaces. Cross-checking the data, the prime suspect turned out to be the **sky**, present in about half of the class's training photos. Trunks and palmera cola de pescado were already problematic. |
| Test 2 | v2 | 126 | The object problem was **resolved**: objects now almost always end up in `other`. `Drago`, `olivo` and `palmera canaria` are recognized from **full-view shots**. Isolated trunks and the palmera cola de pescado (5/22) remain weak. |
| Test 3 | v3 | — | Generic trunks correctly rejected, but `palmera canaria` recognized 1 time out of 26. Version discarded. |

**Total: 220 test photographs**, in addition to the training ones.

Recurring observations from the tests:

- **Full-view shots work, close-ups don't.** The exception is the palmera cola de pescado,
  for which close-ups of the leaves give better results than the full shot.
- **Isolated trunks are the general weak spot**, shared by several classes.
- `mejorana` gets confused with other very similar small green plants: accepted behavior,
  not a labeling error.
- `alcalifa` is poorly recognized when plants of a different color are present in the frame.

---

## Online demo

The model is published not as a bare classifier, but as a **botanical treasure hunt**:
the user photographs the plants in the park and the model confirms whether they have found one
of the six species. Finding all of them unlocks a shareable summary image.

- **Link:** https://planthuntsanabria.netlify.app/
- **Platform:** Netlify. The model runs **entirely in the browser** via **ONNX**
  (onnxruntime-web): no server, no photos uploaded or stored anywhere.
- **Model used:** v2 (7 categories).
- **Manual confirmation button:** given the documented difficulty with the palmera cola de
  pescado, after a failed attempt the user can manually confirm they are standing in front of
  that plant. This is a deliberate choice limited to that single species: it prevents the game
  from getting stuck on a known limitation of the model, and it is openly stated on the site.

The page states that it is an educational project, reports the real numbers (206 training
photos, 220 test photos, ~88%) and explains which shots work best, so that the user is
guided rather than left to guess.

---

## Repository structure

- `notebook.ipynb` — training notebook (data preparation, training, evaluation).
- `registro_allenamenti.md` — table of all experiments for each version, with
  confusion matrix readings.
- `diario_decisioni.md` — the project's methodological choices and their rationale (decision log),
  including the raw results of the field tests.
- `README.md` — this file.

**Not** included in the repository (data and models should not be versioned on GitHub):

- The dataset images (v1, v2, v3) → stored separately; all versions are kept
  for reproducibility.
- The trained models (`.pkl` / `.onnx`) → stored separately.

---

## Known limitations

An educational project with a small dataset; the results should be read honestly:

- **Small validation set** (35–41 photos depending on the version): accuracy is indicative but
  statistically noisy, and not comparable across versions.
- **Few images per class:** room for improvement by collecting more data, especially
  for the weakest classes (palmera cola de pescado first and foremost).
- **Sensitivity to background and partial framing** (isolated trunks): partly addressed
  in v2, not yet solved.
- **Palmera cola de pescado:** unreliably recognized (5/22 in the field). Its thin stems
  blend in with the surrounding vegetation. Mitigated in the game with manual confirmation,
  not solved in the model.
- Some confusions reflect genuine visual similarity between species, not labeling errors.
- **Changes not isolated:** the v2 corrections were all introduced at once, so the
  "sky → drago" hypothesis remains plausible but unproven. A more rigorous future cycle
  would change one variable at a time, replacing only the `drago` class in v1 with the one
  from v2. All datasets are kept precisely to make this verification possible.

---

## Next steps

- Verify in isolation the uniform-background hypothesis for the `drago` class.
- Collect more trunk photos for the classes that have few (`drago`, `olivo`) and
  rebalance the `other` class, which has too many.
- Rethink the strategy for the palmera cola de pescado: more full-view shots in different
  settings, or redefine the use case to leaf close-ups only.
- Replace the random train/validation split with a **per-specimen** split, to prevent
  near-duplicate shots from ending up on both sides.

---

## A note on transparency

The Python scripts used to batch-rename the dataset files were developed with the help of
an AI tutor, as was the demo website. They were used with a full understanding of their
purpose, and for this reason the scripts are not included in the repository. The conceptual
preparation of the datasets (selection, label verification, discard criteria), the design
and execution of the field tests, the error diagnosis and the analysis of the results are
entirely my own work.

---

## License

Code released under the **MIT** license (see [`LICENSE`](LICENSE)).
Dataset images, when published, will have their own license stated on their page.
