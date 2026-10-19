# Execution, recovery and reproducibility

PuranOS separates collaborative project state from execution state and engineering results. This allows a long-running task to retain its context without treating every conversation as a new source of truth.

## Coordination and execution

OpenProject carries work packages, dependencies and human-visible task context. The communication runtime routes work to configured specialists. PostgreSQL-backed execution records retain work items, attempts, state transitions and side-effect history. Hatchet supports durable and scheduled workflows where configured.

Deduplication and recovery behavior belong to the particular workflow. A durable queue does not guarantee that every external action is safe to repeat. A mutation with an uncertain result needs an explicit recovery path.

## Calculation runner

The shared engineering calculation runner retains attempt information and stage artifacts. It can resume verified work, while changes to relevant inputs require replay rather than silently reusing an obsolete result. A stage that may already have persisted engineering state needs its defined recovery behavior.

## Model code pins

Engineering sizing series can pin reached model code to a recorded source revision. The implementation checks the code used by the calculation, including reached model/kernel code paths. Run receipts retain the relevant pin and execution context.

Runtime library versions are recorded and changes are reported. This is a warning about reproducibility, not a claim that dependency drift is universally blocked. Older series may acquire their library baseline only when first run through the newer receipt path; that earlier interval cannot be retrospectively verified.

Fixed code alone does not prove an engineering result. Inputs, solver behavior, model configuration and applicability still matter.

## Authority and learning

Task policy and server-side checks constrain actions where implemented. Persona text is not the only security boundary. Engineering preparation, software checks and release are separate steps.

A reviewed correction enters shared tools, tests or instructions. An existing project adopts it through a new revision or an explicitly replayed calculation. This retains the distinction between what was originally calculated and what is known now.

[Source snapshot](../../source-snapshot.json) · [Specialist handoffs](../personas/README.md)
