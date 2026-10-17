# /new-project Slash Command

Execute the new project setup intake and generation pipeline.

## Usage
Run `/new-project` in Claude Code to initiate the step-wise intake interview.

## Agent Execution Instructions
When this command is triggered:
1. Ask the user for the target folder path where the project should be scaffolded.
2. Load `AGENTS.md` and initiate the intake sequence from `intake/step-wise-intake.md`.
3. Restrict options dynamically based on preceding answers using `intake/decision-tree.md`.
4. Halt with the `[NOT YET SUPPORTED]` banner if a Tier 3 choice is requested.
5. Output the populated `project-spec.yaml` and wait for human confirmation.
6. Generate the generation plan with `shared-utils`, declarative routes, functional controllers, `validateData(schema)` route middleware, `APIResponseEntity<T>`, and no test folders, wait for approval, and execute [new-project.md](../../workflows/new-project.md).
