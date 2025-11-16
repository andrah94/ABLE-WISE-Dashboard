# CLAUDE.md - AI Assistant Guide for ABLE-WISE-Dashboard

## Project Overview

**Project Name:** ABLE & WISE Platform Dashboard
**Repository:** ABLE-WISE-Dashboard
**Type:** Single-page web application
**Tech Stack:** Vanilla HTML, CSS (Tailwind CDN), JavaScript
**Purpose:** Task management and workflow automation platform for two organizations:
- ABLE Foundation (nonprofit/foundation)
- WISE Financial Partners (financial services)

## Architecture & File Structure

```
ABLE-WISE-Dashboard/
├── index.html          # Main and only application file (contains HTML, CSS, and JS)
├── CLAUDE.md           # This file - AI assistant documentation
└── .git/               # Git repository data
```

### Single-File Architecture

This application uses a **monolithic single-file architecture** where everything is contained in `index.html`:

1. **HTML Structure** (lines 1-221): Complete page markup
2. **CSS Styles** (lines 11-219): Embedded in `<style>` tag
3. **JavaScript Logic** (lines 397-704): Embedded in `<script>` tag

**Important:** All changes to functionality, styling, or markup happen in `index.html`.

## Application Structure

### Three Main Views

The application has three primary view containers that users can switch between:

1. **Main View** (`#mainView`) - Lines 239-274
   - Aggregated dashboard showing all tasks from both organizations
   - Daily planner with 7-day view
   - Master task list with filtering and sorting
   - Three viewing modes: List, Calendar, Timeline

2. **ABLE Foundation View** (`#ableView`) - Lines 276-307
   - Black and silver color scheme
   - Kanban board with statuses: Todo, In Progress, Completed
   - Task creation form
   - Filter/search panel
   - Custom workflow builder

3. **WISE Financial View** (`#wiseView`) - Lines 309-360
   - Dark background with gold accents
   - Kanban board with statuses: Inbox, Assigned, In Progress, Complete
   - Task creation form
   - Workflow trigger form
   - Quick notes panel
   - Custom workflow builder

### Color Scheme & Branding

**CSS Variables (lines 13-28):**
```css
--main-bg: #18181B          /* Zinc-900 for main view */
--able-bg: #000000          /* Pure black for ABLE */
--able-accent: #C0C0C0      /* Silver accent */
--wise-bg: #111111          /* Near-black for WISE */
--wise-accent: #FFD700      /* Gold accent */
```

**Design Principle:** Each organization has distinct visual identity while maintaining cohesive UX.

## Data Model

### AppData Object (lines 407-413)

Central data store holding all application state:

```javascript
{
  mainTasks: [],          // Generic/unassigned tasks
  ableTasks: [],          // ABLE Foundation tasks
  wiseTasks: [],          // WISE Financial tasks
  ableWorkflows: [],      // ABLE custom workflows
  wiseWorkflows: [],      // WISE custom workflows
  wiseQuickNotes: [],     // WISE quick notes
  currentSort: {},        // Master table sort state
  mainCalendar: {}        // Calendar view state
}
```

### Task Object Schema

```javascript
{
  id: string,             // Generated via Utils.generateId()
  title: string,          // Task title
  dueDate: string,        // YYYY-MM-DD format
  assignee: string,       // Person assigned
  notes: string,          // Additional notes
  recurring: string,      // 'none' | 'daily' | 'weekly' | 'monthly'
  status: string,         // Varies by organization (see below)
  organization: string    // 'main' | 'able' | 'wise'
}
```

**Status Values:**
- ABLE: `todo`, `inprogress`, `completed`
- WISE: `inbox`, `assigned`, `inprogress`, `complete`
- Main View: Uses organization-specific statuses

### Workflow Object Schema

```javascript
{
  id: string,
  name: string,
  steps: [{
    name: string,
    assignee: string
  }],
  organization: string    // 'able' | 'wise'
}
```

## Key Components & Functions

### DOM Element References (lines 416-480)

All DOM elements are cached in the `DOM` object for performance. Access elements via `DOM.elementName` instead of `document.getElementById()`.

### Core Utility Functions (lines 399-404)

**Utils Object:**
- `generateId()` - Creates unique task IDs
- `getCurrentDate()` - Returns today's date in YYYY-MM-DD format
- `getDaysArray(start, count)` - Generates array of dates
- `formatDate(date, options)` - Formats dates for display

### View Management (lines 483-510)

**switchView(targetViewId):**
- Handles page transitions with animations
- Updates navbar active state
- Triggers ABLE section header animations
- Manages logo visibility

**Animation Classes:**
- `slide-in-from-right` / `slide-in-from-left`
- `slide-out-to-left` / `slide-out-to-right`
- `fadeIn`

### Drag-and-Drop System (lines 536-548)

**Key Functions:**
- `handleDragStart(e)` - Initiates drag
- `handleDrop(e, taskListSelector)` - Handles drop and updates task status/date
- `setupDragAndDropListeners(containerSelector, columnSelector, taskListSelector)` - Initializes D&D

**Behavior:**
- Tasks can be dragged between Kanban columns (changes status)
- Tasks can be dragged between daily planner days (changes due date)
- Supports both ABLE and WISE boards
- Automatically re-renders affected views after drop

### Rendering Functions

**Main View:**
- `renderMainPlanner()` - 7-day daily planner (lines 551-553)
- `renderMasterTaskList()` - Sortable/filterable task table (lines 557-559)
- `renderMainCalendar()` - Monthly calendar grid (lines 569-571)
- `renderMainTimeline()` - Gantt-style timeline (lines 573-575)

**ABLE View:**
- `renderAbleTasks()` - Renders ABLE Kanban board (lines 581-583)

**WISE View:**
- `renderWiseTasks()` - Renders WISE Kanban board (lines 622-624)
- `renderWiseQuickNotes()` - Renders quick notes list (lines 628-630)

### Task Management Functions

**Adding Tasks:**
- `addAbleTask()` - Creates ABLE task from form (lines 578-580)
- `addWiseTask()` - Creates WISE task from form (lines 595-617)

**Note:** Tasks are added to respective arrays and all views are re-rendered.

**Editing Tasks:**
- `editTaskPrompt(taskId, taskOrg)` - Simple prompt-based editing (lines 560-562)

**Filtering & Sorting:**
- `sortMasterTable(column)` - Toggles sort order (lines 563-565)
- Filters applied in render functions based on input values

### Workflow Management

**ABLE Workflows (lines 584-592):**
- `openAbleWorkflowBuilder()` - Opens workflow modal
- `addAbleWorkflowStepRow()` - Adds step input fields
- `saveAbleWorkflow()` - Saves workflow to AppData

**WISE Workflows (lines 633-662):**
- `openWiseWorkflowBuilder()` - Opens workflow modal
- `addWiseWorkflowStepRow()` - Adds step input fields
- `saveWiseWorkflow()` - Saves workflow to AppData
- `triggerWiseWorkflow()` - Creates tasks from predefined workflow (lines 619-621)

### Modal System (lines 513-522)

**Functions:**
- `openModal(modalId, title, bodyContent, footerContent)`
- `closeModal(modalId)`

**Available Modals:**
- `genericModal` - General-purpose modal
- `ableWorkflowModal` - ABLE workflow builder
- `wiseWorkflowModal` - WISE workflow builder

## Development Workflows

### Making UI Changes

1. **Locate the view** in HTML (lines 239-360)
2. **Update markup** as needed
3. **Adjust styles** in the `<style>` section (lines 11-219)
4. **Test across all three views** to ensure consistency

### Adding New Task Fields

1. **Update HTML form inputs** in respective view sections
2. **Modify task object creation** in `addAbleTask()` or `addWiseTask()`
3. **Update task card rendering** in `createTaskCardElement()` (lines 525-533)
4. **Update edit functionality** in `editTaskPrompt()`

### Adding New Features

1. **Add data structures** to `AppData` object
2. **Create render function** following naming convention: `render[Feature]Name()`
3. **Add event listeners** in DOMContentLoaded (lines 671-701)
4. **Update relevant views** to display new feature
5. **Test drag-and-drop compatibility** if feature involves tasks

### Modifying Filters

1. **Add input elements** to filter panels
2. **Cache DOM references** in `DOM` object
3. **Modify filter logic** in relevant render functions
4. **Add event listener** for live filtering

## Code Conventions & Patterns

### Naming Conventions

- **Global Objects:** PascalCase (`AppData`, `DOM`, `Utils`)
- **Functions:** camelCase (`renderMainPlanner`, `addAbleTask`)
- **CSS Classes:** kebab-case (`task-card`, `kanban-column`)
- **Data Attributes:** camelCase (`data-taskId`, `data-view`)
- **CSS Variables:** kebab-case (`--main-bg`, `--able-accent`)

### Organization-Specific Prefixes

- **ABLE:** Functions and elements prefixed with `able` (e.g., `ableAddTaskButton`, `addAbleTask()`)
- **WISE:** Functions and elements prefixed with `wise` (e.g., `wiseKanbanBoard`, `renderWiseTasks()`)
- **Main:** Functions prefixed with `main` or `master` (e.g., `renderMainPlanner`, `masterTaskTable`)

### Event Handling Pattern

```javascript
// 1. Cache DOM element reference
const element = document.getElementById('elementId');

// 2. Define handler function
function handleEvent() {
  // Implementation
}

// 3. Attach listener in DOMContentLoaded
document.addEventListener('DOMContentLoaded', () => {
  element.addEventListener('click', handleEvent);
});
```

### Rendering Pattern

All render functions follow this pattern:

```javascript
function renderSomething() {
  // 1. Clear existing content
  container.innerHTML = '';

  // 2. Apply filters if applicable
  let data = getData();
  data = applyFilters(data);

  // 3. Loop and create elements
  data.forEach(item => {
    const element = createElementForItem(item);
    container.appendChild(element);
  });

  // 4. Reinitialize interactions (D&D, etc.)
  setupDragAndDropListeners(...);
}
```

### State Management Pattern

- **Single source of truth:** All data lives in `AppData`
- **Immutability not enforced:** Direct mutation is used
- **Render on change:** After data changes, call all affected render functions
- **No persistence:** Data resets on page reload (no localStorage/backend)

### Task Creation Pattern

```javascript
function addOrganizationTask() {
  // 1. Validate required fields
  const title = DOM.taskTitle.value.trim();
  if (!title) { alert("Title required."); return; }

  // 2. Create task object
  const newTask = {
    id: Utils.generateId(),
    // ... other fields
    organization: 'able' // or 'wise'
  };

  // 3. Add to data store
  AppData.organizationTasks.push(newTask);

  // 4. Re-render affected views
  renderOrganizationTasks();
  renderMasterTaskList();
  renderMainPlanner();
  renderMainCalendar();
  renderMainTimeline();

  // 5. Clear form
  DOM.taskTitle.value = '';
  // ... clear other fields
}
```

## Important Considerations

### Performance

- **DOM references are cached** to avoid repeated queries
- **Render functions rebuild entire views** - consider optimization for large datasets
- **No virtualization** - all tasks render regardless of visibility

### Browser Compatibility

- Uses **modern JavaScript** (ES6+): arrow functions, template literals, const/let
- Requires **CSS Grid and Flexbox** support
- **Drag and Drop API** must be supported
- **Font Awesome 6.5.1** and **Tailwind CSS CDN** loaded from CDN

### External Dependencies

1. **Tailwind CSS** - `https://cdn.tailwindcss.com`
2. **Google Fonts:**
   - Inter (primary UI font)
   - Lora (ABLE Foundation)
   - Roboto (WISE Financial)
3. **Font Awesome 6.5.1** - Icons

### Data Persistence

**Current State:** NO PERSISTENCE
- All data is lost on page reload
- AppData exists only in memory
- No localStorage, sessionStorage, or backend integration

**To Add Persistence:**
1. Implement `saveToLocalStorage()` function
2. Call after each data mutation
3. Implement `loadFromLocalStorage()` function
4. Call in DOMContentLoaded before initial render

### Known Limitations

1. **No backend integration** - purely client-side
2. **No user authentication** - admin icon is decorative
3. **No real workflow execution** - workflows saved but not executed
4. **Simple edit functionality** - uses browser prompts, not inline editing
5. **No task deletion** - feature not implemented
6. **No data validation** - minimal client-side validation
7. **No error handling** - limited try-catch blocks

## Testing Approach

### Manual Testing Checklist

**View Switching:**
- [ ] All three views render correctly
- [ ] Navbar updates active state
- [ ] Transitions animate smoothly
- [ ] Logos toggle appropriately

**Task Creation:**
- [ ] ABLE tasks appear in ABLE board and main views
- [ ] WISE tasks appear in WISE board and main views
- [ ] Tasks respect organization colors
- [ ] Form fields clear after submission

**Drag and Drop:**
- [ ] Tasks drag within same organization
- [ ] Status updates when dropped in new column
- [ ] Due date updates when dropped in planner day
- [ ] All views update after drop

**Filtering & Sorting:**
- [ ] Master task table filters work
- [ ] Master task table sorting works
- [ ] ABLE filters work
- [ ] Calendar and timeline display correct tasks

**Workflows:**
- [ ] ABLE workflow builder saves workflows
- [ ] WISE workflow builder saves workflows
- [ ] WISE workflow trigger creates tasks
- [ ] Workflow steps can be added

**Quick Notes:**
- [ ] Notes can be added
- [ ] Most recent 5 notes display
- [ ] Timestamps are correct

## Common Modification Patterns

### Adding a New View

```javascript
// 1. Add HTML section
<section id="newView" class="page-container">
  <!-- Content -->
</section>

// 2. Add nav button
<button data-view="newView">New View</button>

// 3. Update switchView logic if needed
// 4. Add initialization in DOMContentLoaded
```

### Adding a New Task Status

```javascript
// 1. Add column to Kanban board HTML
<div class="kanban-column" data-status="newStatus">
  <h3>New Status</h3>
  <div class="task-list min-h-[200px]"></div>
</div>

// 2. Update task creation to use new status
// 3. Drag and drop will automatically work
```

### Styling Organization-Specific Elements

```css
/* ABLE: Black and silver */
#ableView .element {
  background-color: var(--able-bg);
  color: var(--able-text-on-black);
  border-color: var(--able-accent);
}

/* WISE: Dark and gold */
#wiseView .element {
  background-color: var(--wise-bg);
  color: var(--wise-text);
  border-color: var(--wise-accent);
}
```

### Adding Global Filters

```javascript
// 1. Add filter input to HTML
<input id="newFilter" type="text" placeholder="Filter...">

// 2. Cache in DOM object
DOM.newFilter = document.getElementById('newFilter');

// 3. Apply in render function
let filtered = tasks.filter(t =>
  t.someField.toLowerCase().includes(DOM.newFilter.value.toLowerCase())
);

// 4. Add event listener
DOM.newFilter.addEventListener('input', renderFunction);
```

## Git Workflow

### Branch Strategy

- **Feature branches:** Named with `claude/` prefix (e.g., `claude/claude-md-mi15btct6aw0xxep-012x5JUM49DreGaNQJjfJhFG`)
- **Main branch:** Not specified in current git status
- **Current branch:** `claude/claude-md-mi15btct6aw0xxep-012x5JUM49DreGaNQJjfJhFG`

### Commit Guidelines

- Write clear, descriptive commit messages
- Focus on "why" rather than "what"
- Group related changes together
- Test before committing

### Push Guidelines

- Always use: `git push -u origin <branch-name>`
- Branch must start with `claude/` and match session ID
- Retry up to 4 times with exponential backoff on network errors

## Best Practices for AI Assistants

### When Making Changes

1. **Read index.html first** - Understand current state
2. **Test in context** - Consider impact on all three views
3. **Maintain consistency** - Follow existing patterns
4. **Update all views** - Changes to tasks affect multiple renders
5. **Preserve styling** - Respect organization color schemes

### Communication Guidelines

- Reference code by line numbers: `index.html:123`
- Explain impact on multiple views when relevant
- Note any breaking changes to existing functionality
- Suggest testing steps after modifications

### Avoid Common Pitfalls

1. **Don't create new files** - Everything goes in index.html
2. **Don't break CSS variables** - Many elements depend on them
3. **Don't forget to re-render** - UI won't update without render calls
4. **Don't mix organization styles** - Keep ABLE and WISE distinct
5. **Don't skip event listeners** - New elements need interaction setup

### Recommended Change Order

1. Update data structures (AppData)
2. Update HTML markup
3. Update CSS styles
4. Update JavaScript logic
5. Update render functions
6. Add event listeners
7. Test all three views

## Troubleshooting Guide

### Tasks Not Appearing

- Check if task added to correct array (`ableTasks`, `wiseTasks`, `mainTasks`)
- Verify render function called after data change
- Check filter values aren't hiding tasks
- Verify organization field matches view

### Drag and Drop Not Working

- Ensure `setupDragAndDropListeners()` called after render
- Verify `data-status` or `data-date` attributes on columns
- Check if `draggedItem` is properly set in `handleDragStart()`
- Confirm task list selector matches HTML structure

### Styling Issues

- Check if CSS variable exists and has correct value
- Verify element is in correct view context (`#ableView`, `#wiseView`, `#mainView`)
- Look for conflicting Tailwind classes
- Check if styles are scoped to correct view

### Modal Issues

- Verify modal ID matches in HTML and JavaScript
- Check if `closeModal()` is properly called
- Ensure modal has `modal` class and structure
- Verify modal content is being set before opening

## Future Enhancement Ideas

1. **Data Persistence:** Add localStorage or backend integration
2. **Task Deletion:** Implement delete functionality
3. **Inline Editing:** Replace prompts with inline editors
4. **Batch Operations:** Select and modify multiple tasks
5. **Export/Import:** JSON export for backup
6. **Notifications:** Due date reminders
7. **Search:** Global search across all tasks
8. **Analytics:** Dashboard with charts and metrics
9. **Workflow Execution:** Actually run saved workflows
10. **Collaboration:** Real-time updates, comments, attachments

---

**Last Updated:** 2025-11-16
**Version:** 1.0
**Maintained for:** AI Assistants (Claude Code)
