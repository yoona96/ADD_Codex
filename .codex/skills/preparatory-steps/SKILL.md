---
name: preparatory-steps
description: This is a skill for step-by-step process following the ADD 3.0 preparatory steps.
---

# Preparatory steps

# Preparation step 1: Review inputs
- Please review the `/HPS/architectural-driver.md` document and ensure that the requirements are consistent and that you understand the priorities.
- Ask clarification questions, if needed.

## Preparation step 2: Create domain model
- Consider the `/HPS/architectural-driver.md` document and create a domain model for the system.
- Create this domain model in the `/HPS/design/architecture.md` document.
- Represent the domain model using a class diagram using mermaid format.
- Include a table that describes each element of the domain model.

## Preparation step 3: Create iteration plan
- Consider the requirement priorities in `/HPS/architectural-driver.md` and the domain model in `/HPS/design/architecture.md`.
- Create an iteration plan for the design of the system, according to the ADD process.
- Describe, for each iteration, what the main goal will be.
    - The first iteration should be focused on initially structuring the system.
    - The following iterations should be focused on addressing high-priority drivers.
- Be sure to address requirements that directly support the business in early iterations.
- Present the iteration plan as a table with iteration number, goal and list of drivers to be addressed.
- Output the iteration plan to an `/HPS/design/iteration-plan.md` file. Follow `template.md` when generating the iteration plan.