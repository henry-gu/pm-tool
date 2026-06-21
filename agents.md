# AGENTS.md

## Project goal

Build a pure HTML/CSS/JavaScript project management web app that mimics core Microsoft Project-style planning features. The app must run locally in a browser and manage project plan data stored in CSV files.

## Core constraints

* Use plain HTML, CSS, and JavaScript only.
* No backend server required for the main version.
* No build system unless absolutely necessary.
* Prefer a single-page app with separate files:

  * `index.html`
  * `styles.css`
  * `app.js`
  * `sample-project.csv`
  * `README.md`
* The browser cannot silently overwrite local CSV files. Implement CSV import/export using file picker and download. If supported, optionally use File System Access API for “open/save” in Chromium browsers.
* Keep code readable and heavily commented.

## Data model

Project CSV columns must include at least:

* `id`
* `name`
* `level`
* `type`
* `status`
* `duration`
* `start_date`
* `end_date`
* `dependencies`
* `owner`
* `resources`
* `percent_complete`
* `priority`
* `notes`

Support extra unknown CSV columns without deleting them.

## Required views

Implement these views:

1. Task Table View

   * Editable spreadsheet-like grid.
   * Add, delete, duplicate, indent, outdent, reorder tasks.
   * Validate task IDs, dates, duration, and dependencies.

2. Gantt Chart View

   * Timeline bars based on start and end date.
   * Dependency lines or clear dependency indicators.
   * Status/progress display.
   * Zoom by day/week/month.

3. Kanban / Status View

   * Columns for not started, in progress, completed, blocked.
   * Drag tasks between statuses.

4. Owner / Resource View

   * Group tasks by owner and resources.
   * Show workload summary.

5. Calendar / Milestone View

   * Show tasks and milestones by date.

6. Dashboard View

   * Total tasks.
   * Completed / in-progress / not-started count.
   * Overdue tasks.
   * Upcoming tasks.
   * Progress summary.

## User roles

Support two practical user modes:

* Project Manager:

  * Can edit all fields.
  * Can add, delete, reorder, import, export, and recalculate.

* Task Owner:

  * Can update status, percent complete, notes, and actual dates.
  * Should not accidentally change structure or dependencies.

Use a simple UI toggle for role mode. Do not implement authentication.

## Functional requirements

* Import CSV.
* Export CSV.
* Create a new project from a sample template.
* Auto-calculate end date from start date and duration where possible.
* Detect dependency issues:

  * Missing dependency IDs.
  * Circular dependencies.
  * Tasks starting before dependencies finish.
* Search and filter tasks.
* Sort by key fields.
* Preserve hierarchy using the `level` field.
* Support collapsible task groups.
* Save draft data in `localStorage`.
* Warn before losing unsaved changes.

## UI requirements

* Make the app professional and usable by project managers.
* Use responsive layout.
* Use accessible controls and semantic HTML.
* Avoid heavy visual clutter.
* Include keyboard-friendly table editing where practical.
* Provide clear empty states and validation messages.

## Implementation approach

Build iteratively:

1. Create the project scaffold and sample CSV.
2. Implement CSV parser/stringifier.
3. Implement internal task data model.
4. Implement table view editing.
5. Implement import/export.
6. Implement dashboard metrics.
7. Implement Gantt view.
8. Implement Kanban view.
9. Implement resource and calendar views.
10. Add validation, localStorage autosave, and README.

## Testing expectations

Before finishing, verify:

* App opens from `index.html`.
* Sample CSV loads correctly.
* Imported CSV can be edited and exported.
* Exported CSV can be re-imported without losing data.
* Gantt dates align with task start/end dates.
* Dependency validation works.
* Role toggle changes editing permissions.
* No console errors in normal usage.

## README requirements

Create a README with:

* What the app does.
* How to open it.
* Browser file read/write limitations.
* CSV schema.
* Sample workflow.
* Known limitations.
* Future enhancement ideas.

## Definition of done

The result is done when a user can open `index.html`, load or create a CSV-based project plan, edit it through multiple views, view a Gantt chart and dashboard, then export the updated project CSV.
