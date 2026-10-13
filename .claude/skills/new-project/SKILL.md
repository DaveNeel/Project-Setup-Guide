---
name: new-project
description: Guides the agent through step-by-step project intake and backend generation following architectural standards.
---

# New Project Generation Skill

## Purpose
Enables Claude Code to interactively interview a developer, record project decisions into a formal specification, validate against compatibility constraints, and generate a production-ready backend project adhering to the AI-Ready Project Standards.

## Execution Sequence
1. **Target Directory First:**
   - Prompt the user: *"What folder path should the new backend project be generated in?"*
2. **Step-Wise Intake:**
   - Read `AGENTS.md` and `intake/step-wise-intake.md`.
   - Ask the user questions Step 1 through 10 one at a time.
   - Present only options valid for prior answers.
3. **Compile & Validate:**
   - Validate answers against `intake/project-spec.schema.yaml`, `intake/decision-tree.md`, and `intake/compatibility-matrix.md`.
   - Render the complete `project-spec.yaml` to the user and request confirmation.
4. **Plan & Confirm:**
   - Consult the Standards Routing Table in `AGENTS.md` to load only matching standards.
   - Present the detailed generation plan (folder layout, dependencies, ports, env keys).
   - Ensure `shared-utils` with enums, `validateData(schema)` route middleware, `APIResponseEntity<T>` envelope, declarative `registerRoutes`, `handleRequestWithResponse`, and single dynamic `/health` probe are specified.
   - Omit `test/` folders from the baseline setup.
   - Wait for explicit user approval before writing code.
5. **Scaffold & Build:**
   - Follow `workflows/new-project.md` to scaffold into the target external directory.
   - Run `npm install`, `npm run build`, and `npm run lint` in the target project.
   - Fix any failures until all commands pass cleanly.
6. **Audit & Report:**
   - Evaluate against `checklists/conformance-checklist.md`.
   - Output the completed handoff report and stamp the target `README.md` with standards `VERSION`.
