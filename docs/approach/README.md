# Why PuranOS is built this way

The engineering service should leave behind information that the next person or agent can use. That objective leads to several practical choices.

## Retain the basis

Use structured records for equipment identity, source inputs, assumptions and result metadata. Use linked narrative knowledge for rationale that does not fit a field. Neither a chat transcript nor a search result should silently replace the approved project basis.

## Give specialists defined responsibilities

Process, CAD, deliverables, finance and maintenance work need different tools and review questions. Bounded roles make the handoff inspectable. Parallel preparation is useful when it respects shared-state ownership and dependencies.

## Put engineering methods in tools

An agent can frame a calculation and prepare inputs. The engineering tool implements a defined model or method. Applicability, convergence, units and source evidence remain review questions. See [engineering computation](engineering-engines.md).

## Deliver the information with the document

Documents and spreadsheets are useful human interfaces. Structured records make those outputs usable in a customer’s subsequent workflows, including their own agents. See [deliverable integration](../integration/deliverables.md).

## Keep reviewed corrections available

Shared tools and skills retain improvements discovered through work. Changes are versioned and reviewed; an existing project adopts them deliberately. See [skills and reviewed learning](skills-as-expertise.md).

## Enforce authority at the action

Instructions describe expected behavior. Deployment permissions, runtime policy and the actual tool implementation determine what can be accessed or changed. A software check passing is not engineering approval.

The linked research and historical design essays provide background. The [current capability notes](../current-state.md) define the scope of the public description of today’s implementation.
