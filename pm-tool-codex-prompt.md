# Codex Command Prompts — Project Management Tool

Use these prompts in sequence in **Codex Desktop** or **Codex CLI**.
Each prompt is self-contained. Run them in order for best results.

---

## Initial Setup Prompt (paste this first)

```
Read AGENTS.md thoroughly. You are building a pure HTML/CSS/JS project management 
web application — a single file called index.html — that mimics Microsoft Project. 
No external JS libraries. No build tools. No backend. CSV file for data storage.

Before writing any code, confirm you understand:
1. The five views required (Gantt, Table, Board, Timeline, Dashboard)
2. The CSV column schema
3. The file I/O strategy (File System Access API + fallback)
4. The implementation order listed at the bottom of AGENTS.md

Summarise the architecture in 10 bullet points, then wait for my go-ahead.
```

---

## Step 1 — HTML Shell + CSS

```
Following AGENTS.md, build Step 1: the HTML skeleton.

Create index.html with:
- The complete <head> with Inter font from Google Fonts and all CSS custom properties 
  (colours, typography scale, spacing) defined as variables on :root
- Top bar (48px, dark navy #1A1F2E) with: app logo/name "ProjectPlan", project title 
  placeholder, and three buttons: Open, Save, Save As
- Toolbar (40px, white) with: five view toggle buttons (Gantt/Table/Board/Timeline/Dashboard) 
  as a pill group, a search input, a Filter button with badge, a "+ Add Task" button
- Main content area (#main-content) that fills all remaining viewport height
- Status bar (28px, #F0F2F5) showing task counts and filename
- A toast notification container (fixed, bottom-right)
- All CSS written in a single <style> tag — layout, component styles, status colours, 
  priority colours as listed in AGENTS.md

Do not add any JavaScript yet. The page should render the chrome perfectly in a browser.
```

---

## Step 2 — CSV Parser & Data Model

```
Add to index.html — Step 2: CSV parser and data model (inside <script type="module">).

Implement:
1. parseCSV(text) → Task[] 
   - Handles quoted fields (especially the dependencies field which has commas)
   - Parses dependencies into string array, resources into string array
   - Parses percent_complete as integer, duration_days as integer, milestone as boolean
   - Adds computed fields: _depth (from dot count in task_id), _isGroup (task_level !== 'task'), 
     _children (populated by buildTree()), _parent (parent task_id)

2. serialiseCSV(tasks) → string
   - Writes all columns in the exact order defined in AGENTS.md
   - Properly quotes fields that contain commas (dependencies, resources)
   - Outputs a header row

3. buildTree(tasks) → tasks (with _children and _parent populated)

4. The state object as defined in AGENTS.md

Test with this inline sample (add as a JS const and run parseCSV on it, log to console):
task_id,task_name,task_level,task_status,duration_days,start_date,end_date,dependencies,owner,resources,percent_complete,notes,priority,milestone
1,Test Project,project,in_progress,30,2025-01-06,2025-02-04,,Alice,,50,,high,false
1.1,Phase 1,group1,completed,14,2025-01-06,2025-01-19,,Alice,,100,,high,false
1.1.1,Task A,task,completed,5,2025-01-06,2025-01-10,,Bob,"Bob,Carol",100,,medium,false
1.1.2,Task B,task,in_progress,5,2025-01-13,2025-01-17,1.1.1,Alice,,60,,high,false
```

---

## Step 3 — File I/O

```
Add to index.html — Step 3: File I/O functions.

Implement these functions:
1. openFile()
   - Try window.showOpenFilePicker with CSV filter
   - On success: store fileHandle in state.project.fileHandle, read text, call parseCSV, 
     store in state.project.tasks, call buildTree(), update state.project.filename, 
     call renderCurrentView(), updateStatusBar()
   - Fallback (if showOpenFilePicker not available): create hidden <input type="file"> 
     accept=".csv", trigger click, read via FileReader

2. saveFile()
   - If fileHandle exists: get writable, write serialiseCSV(state.project.tasks), close, 
     mark isDirty=false, showToast('Saved ✓', 'success')
   - If no fileHandle: call saveFileAs()
   - Fallback: trigger browser download of the CSV text as a .csv file

3. saveFileAs()
   - Try window.showSaveFilePicker with CSV filter, suggestedName = state.project.filename || 'project.csv'
   - Store new fileHandle, then call saveFile()
   - Fallback: download with prompt-style filename

4. Wire up: Open button → openFile(), Save button → saveFile(), Save As → saveFileAs()
5. Wire up: Ctrl+S → saveFile(), Ctrl+O → openFile()
6. Show "● Unsaved changes" in status bar when isDirty is true

Implement showToast(message, type) — appends a div to #toast-container, removes after 3s.
```

---

## Step 4 — Table View

```
Add to index.html — Step 4: Table View (the simplest full-data view).

Implement renderTable(tasks, ui):
- A full-width scrollable grid with sticky header
- Columns (in order): Task ID, Task Name, Level, Status, Duration, Start Date, End Date, 
  Dependencies, Owner, Resources, % Done, Priority, Milestone, Notes
- task_name column: indent by _depth × 20px, show ▶/▼ chevron for group tasks
- Row striping: alternating #F7F8FA / #FFFFFF
- Status shown as a coloured pill badge (use --status-* colours)
- Priority shown as a coloured dot + text
- Milestone shown as ◆ if true
- Double-click any cell (except task_id) → enter inline edit mode:
  - Replaces cell content with appropriate input (text, number, date, select for status/priority)
  - Enter or blur saves the value to state.project.tasks, marks isDirty=true, re-renders
  - Escape cancels
- Click any row → sets state.ui.selectedTaskId, highlights row, opens task editor panel (Step 5)
- Column header click → sort by that column (toggle asc/desc)
- + Add Task button in toolbar → appends a new blank task row, opens editor

Wire up the Table view button to call switchView('table').
Make switchView() replace #main-content children with the rendered view.
```

---

## Step 5 — Task Editor Side Panel

```
Add to index.html — Step 5: Task Editor side panel (reused by all views).

Build a slide-in panel (#task-editor-panel):
- Fixed position, right: 0, top: 0, height: 100vh, width: 380px
- Starts off-screen: transform: translateX(100%)
- Opens with class 'open' → transform: translateX(0), CSS transition 200ms ease
- Has a close ✕ button (top-right) and a dark overlay behind it

Panel contents (form-like layout, no <form> tag — use divs + buttons):
- Header: "Edit Task" or "New Task" + task_id badge
- Fields (each as a label + input pair):
  - Task Name: text input
  - Task Level: <select> with options (project/group1/group2/group3/task)
  - Status: <select> with options + colour indicator
  - Priority: <select> with colour indicator
  - Owner: text input
  - Resources: text input (comma-separated, shown as tag pills)
  - Start Date: date input
  - End Date: date input
  - Duration: number input (auto-calculated when dates change, editable)
  - % Complete: range slider 0-100 + numeric display
  - Milestone: checkbox
  - Dependencies: multi-select or tag input showing task_id + task_name
  - Notes: textarea

- "Save Changes" button (accent blue) → updates task in state.project.tasks, 
  marks isDirty, closes panel, re-renders current view
- "Delete Task" button (red, outline) → confirm dialog, removes task, re-renders
- Auto-calculate: when start_date or end_date changes, recalculate duration_days
- Show read-only computed fields for groups: "Children: X | Avg Complete: Y%"

Implement openTaskEditor(taskId) and closeTaskEditor().
```

---

## Step 6 — Gantt Chart View

```
Add to index.html — Step 6: Gantt Chart View (most complex view).

Implement renderGantt(tasks, ui):

LEFT PANEL (#gantt-left, width 380px, fixed):
- Sticky header row: Task ID (60px) | Task Name (flex) | Duration (60px) | Start (80px) | 
  End (80px) | Owner (80px) | % (40px)
- Each task row: 32px height, alternating stripe
- task_name: indented by _depth × 20px, chevron ▶/▼ for groups (click to expand/collapse)
- Collapsed groups hide all descendant rows
- Status colour dot before task name
- Click row → openTaskEditor(task.task_id)

RIGHT PANEL (#gantt-right, flex 1, overflow-x: auto):
- Header: time axis showing months/weeks based on ui.ganttZoom
  - month zoom: show month names, with day numbers below
  - week zoom: show week numbers and Mon-Sun
  - quarter/year zoom: show quarters or months only
- Today line: red vertical <div> at correct pixel offset
- Task bars as <div> elements absolutely positioned:
  - left = daysBetween(minDate, task.start_date) × pxPerDay
  - width = task.duration_days × pxPerDay
  - height: 18px, vertically centred in 32px row
  - border-radius: 3px
  - Colour by status (use --status-* colours)
  - Groups: slightly darker, full row width from first to last child
  - Milestones: render as ◆ diamond shape (rotated square div)
  - Hover tooltip: task name, dates, owner, % complete
  - % complete fill: darker shading on left portion of bar
- Dependency arrows as SVG overlay:
  - One <svg> element absolutely positioned covering the entire bars area
  - For each dependency: draw an elbow path from end of predecessor bar to start of 
    successor bar with a right-angle route
  - Arrow colour: #9CA3AF, arrowhead marker
- Critical path tasks: red outline on bar (2px solid #EF4444)

SYNCHRONISATION:
- left and right panels must scroll together vertically (sync scroll events)
- Right panel scrolls independently horizontally

ZOOM CONTROLS in toolbar when Gantt is active:
- Four buttons: Week | Month | Quarter | Year
- Update ui.ganttZoom and re-render

Wire up: Gantt tab → switchView('gantt')
Make 'gantt' the default view on app load (before any file is opened, show empty state message).
```

---

## Step 7 — Kanban Board View

```
Add to index.html — Step 7: Kanban Board View.

Implement renderBoard(tasks, ui):
- Show only leaf-level tasks (task_level === 'task' or tasks with no children)
- Five columns: Not Started | In Progress | On Hold | Completed | Cancelled
- Each column:
  - Header: column name + task count badge
  - Background: light tint of the status colour
  - Scrollable vertically
  - Drop target for drag-and-drop

Each task card:
- White background, 1px border (#E2E5EB), border-radius 8px
- Left accent stripe (4px wide) coloured by priority
- Content: task_id (small, muted), task_name (bold, 13px), owner (small, muted)
- Progress bar: full-width, 4px height, green fill = percent_complete %
- Priority badge: coloured pill (low/medium/high/critical)
- Milestone indicator: ◆ if milestone=true
- Click card → openTaskEditor(task.task_id)

Drag-and-drop (HTML5 native):
- draggable="true" on each card
- dragstart: store task_id in dataTransfer
- dragover on columns: preventDefault, add visual highlight
- drop: update task.task_status to column's status, mark isDirty, re-render board

Filter pills above board: one button per owner (dynamically generated from data).
Click an owner to filter cards to that owner only. Active filter highlighted.
```

---

## Step 8 — Timeline / Roadmap View

```
Add to index.html — Step 8: Timeline / Roadmap (Swimlane) View.

Implement renderTimeline(tasks, ui):
- A swimlane chart grouped by either owner or task_level (toggle button in toolbar)
- Top: time axis (same logic as Gantt, reuse the header code)
- Each swimlane row:
  - Row header (fixed left, 160px): owner name or level name
  - Bars: same rendering as Gantt bars but more compact (height 14px)
  - Multiple bars per swimlane (all tasks for that owner/level)
  - Bars coloured by priority (not status, to differ from Gantt)
  - Milestone diamonds rendered
  - Hover tooltip: task name, status, dates

Group-by toggle: "Group by Owner | Group by Level" — two buttons, active highlighted.
Updates ui.timelineGroupBy and re-renders.

Today line (same as Gantt).
Zoom controls shared with Gantt (reuse ui.ganttZoom).

Wire up: Timeline tab → switchView('timeline')
```

---

## Step 9 — Dashboard View

```
Add to index.html — Step 9: Dashboard / Summary View.

Implement renderDashboard(tasks, ui):

ROW 1 — Summary Cards (5 cards across):
- Total Tasks (count of leaf tasks only)
- Completed (count where status=completed)
- In Progress (count where status=in_progress)  
- Not Started (count where status=not_started)
- Overdue (count where end_date < today AND status != completed AND status != cancelled)
Each card: large number, label below, coloured icon/indicator, coloured left border

ROW 2 — Progress + Key Stats (3 columns):
- Overall % Complete: large donut chart (Canvas) showing weighted average by duration
- Days Remaining: number of days from today to project end_date (from the root task)
- On-time rate: % of completed tasks finished on or before end_date

ROW 3 — Charts (2 columns):
- Left: Vertical bar chart (Canvas): Tasks by Status — 5 bars, one per status, status colours
- Right: Donut chart (Canvas): Tasks by Priority — 4 segments, priority colours, legend below

ROW 4 — Tables (2 columns):
- Left: "Overdue Tasks" table — task name, owner, end_date, days overdue (red)
- Right: "Due This Week" table — task name, owner, end_date, status badge

ROW 5 — Owner Workload Table (full width):
- Columns: Owner | Total Tasks | Completed | In Progress | Not Started | Avg % Complete
- One row per unique owner
- % Complete shown as a mini inline progress bar

Canvas chart helpers to implement:
- drawVerticalBarChart(canvas, {labels, values, colours})
- drawDonutChart(canvas, {segments: [{label, value, colour}]})

Wire up: Dashboard tab → switchView('dashboard')
```

---

## Step 10 — Filtering, Search & Critical Path

```
Add to index.html — Step 10: Filtering, search, and critical path.

SEARCH:
- Search input in toolbar filters by task_name (case-insensitive substring match)
- Updates ui.filters.search on input event
- Applied in getFilteredTasks() before passing to any view renderer

FILTER PANEL:
- "⚙ Filter ▾" button toggles a dropdown panel below toolbar
- Panel contains:
  - Status: checkboxes for each status value
  - Owner: checkboxes for each unique owner (dynamically generated)
  - Priority: checkboxes for each priority value
  - Date range: From / To date inputs
  - Milestone only: checkbox
  - "Clear all filters" link
- Active filter count shown as a blue badge on the Filter button
- All filters stored in ui.filters and applied in getFilteredTasks()

getFilteredTasks():
- Start with state.project.tasks
- Apply search filter
- Apply status, owner, priority, milestone filters
- Apply date range filter (tasks overlapping the date range)
- When a group task matches: show it (its children are separately filtered)
- When a leaf task matches: also show its ancestor group tasks (so hierarchy is intact)
- Return filtered + tree-consistent task list

CRITICAL PATH:
- Implement calculateCriticalPath(tasks):
  - Build a dependency graph
  - Forward pass: earliest start/end for each task
  - Backward pass: latest start/end for each task  
  - Tasks where slack = 0 are on the critical path
  - Return Set of critical task_ids
- In Gantt view: tasks on critical path get a red 2px border on their bar
- In Table view: critical path tasks get a subtle red left border on the row

Detect circular dependencies: if present, show a warning toast and skip critical path.
```

---

## Step 11 — Sample CSV + Final Polish

```
Do two things:

1. CREATE sample_project.csv:
Generate a realistic 28-task project CSV called "Website Redesign Project" with:
- 1 root project task (task_id: 1)
- 3 phase groups at group1 level: Discovery (1.1), Design (1.2), Development (1.3)
- Each phase has 2 sub-groups at group2 level
- Each sub-group has 3-4 leaf tasks
- Start date: 2025-01-06, end date: ~2025-06-30
- Statuses: Discovery phase fully completed, Design phase in progress (~60%), 
  Development phase not started
- 5 owners: Alice Chen, Bob Lee, Carol Wu, David Kim, Emma Singh
- Sequential dependencies within phases, milestone tasks at end of each phase
- Mix of priorities (2 critical, 6 high, 12 medium, 8 low)
- 3 milestone tasks (end of each phase)
- Realistic notes on some tasks

2. FINAL POLISH for index.html:
- Keyboard shortcuts: G/T/B/L/D for views, Ctrl+S, Ctrl+O, Ctrl+F, Escape
- Empty state: when no file is open, show a centred empty-state message in #main-content 
  with "Open a CSV file to get started" and a large Open File button
- Loading state: brief "Loading..." text during CSV parse (for large files)
- Error state: if CSV parse fails, show error toast with the row number
- Confirm dialog before Delete Task
- Responsive: at < 900px width, hide the Gantt left panel column labels that overflow
- Page title: update <title> to show project name when a file is open
- Favicon: use a simple SVG data URI of a Gantt bar icon
- Print CSS: @media print — hide toolbar and top bar, print the current view

Run a final self-review:
- Open browser console — are there any errors?
- Does Gantt row alignment (left panel vs right panel) look correct?
- Do all 5 view tabs work?
- Does save/open round-trip preserve all data?
Report any issues found and fix them.
```

---

## Quick Single-Shot Prompt (Alternative)

If you want Codex to build everything in one go (may require multiple completions):

```
Read AGENTS.md. Build the complete project management web app as a single index.html 
file and a sample_project.csv. Follow the implementation order in AGENTS.md exactly 
(Steps 1-13). The app must:

1. Open and save CSV files via File System Access API (with <input> fallback)
2. Render 5 views: Gantt chart, Table/spreadsheet, Kanban board, Timeline/swimlane, Dashboard
3. Support inline editing, a task editor panel, drag-and-drop in Kanban
4. Show dependency arrows and today-line in Gantt
5. Calculate and highlight the critical path
6. Provide search and multi-filter across all views
7. Use zero external JS libraries

Use the exact colour palette, typography, and layout spec in AGENTS.md.
Start with the HTML skeleton, confirm it renders, then add JS module by module.
Tell me when each step is complete before proceeding to the next.
```

---

## Tips for Using These Prompts in Codex Desktop

1. **Place `AGENTS.md` in your project root** — Codex Desktop automatically reads it
2. **Run prompts in order** — each step builds on the previous
3. **After each step**, open `index.html` in Chrome and verify before continuing
4. **If a step produces errors**, paste the console error back to Codex as a follow-up
5. **Use Codex's "apply to file" feature** — responses will patch `index.html` directly
6. **After Step 11**, do a final test: open `sample_project.csv`, check all 5 views, edit a task, save, re-open

## Tips for Codex CLI

```bash
# Initialise project directory
mkdir project-manager && cd project-manager
# Place AGENTS.md here, then run:
codex "Read AGENTS.md and build Step 1 — HTML skeleton with all CSS"
codex "Add Step 2 — CSV parser and data model to index.html"
# ... and so on
```
