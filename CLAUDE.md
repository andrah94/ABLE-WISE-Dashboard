# CLAUDE.md - AI Assistant Guide for ABLE-WISE-Dashboard

## Project Overview

**Project Name:** ABLE & WISE Platform Dashboard
**Repository:** ABLE-WISE-Dashboard
**Type:** Single-page web application
**Tech Stack:** Vanilla HTML, CSS (Tailwind CDN), JavaScript
**Purpose:** Task management and workflow automation platform for two organizations:
- ABLE Foundation (nonprofit/foundation)
- WISE Financial Partners (financial services)

## Recent Improvements (v2.0)

The application has been significantly elevated with the following enhancements:

### Core Features Added
1. **LocalStorage Persistence** - All data now persists between sessions
2. **Task Deletion** - Delete tasks with confirmation dialogs
3. **Task Duplication** - Copy existing tasks easily
4. **Modal-Based Editing** - Professional edit modal replacing browser prompts
5. **Toast Notifications** - Beautiful, non-intrusive user feedback
6. **Confirmation Dialogs** - Safety checks for destructive actions
7. **Input Validation** - Client-side validation with helpful error messages
8. **Mobile Responsiveness** - Improved layout for mobile devices
9. **Auto-Save** - Automatic data persistence on every change + periodic backup

### User Experience Improvements
- **Professional UI feedback** via toast notifications (success, error, warning, info)
- **Safer operations** with confirmation dialogs for deletions
- **Better task management** with Edit, Copy, and Delete buttons in task table
- **Form validation** prevents invalid data entry
- **Responsive design** works on tablets and phones
- **Data persistence** means work is never lost

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

### Core Utility Functions

**Utils Object:**
- `generateId()` - Creates unique task IDs
- `getCurrentDate()` - Returns today's date in YYYY-MM-DD format
- `getDaysArray(start, count)` - Generates array of dates
- `formatDate(date, options)` - Formats dates for display
- `validateTask(task)` - Validates task object, returns array of errors
- `showToast(message, type)` - Displays toast notification (success, error, warning, info)
- `confirm(title, message)` - Shows confirmation dialog, returns Promise<boolean>

### Toast Notification System

**Usage:**
```javascript
Utils.showToast('Task created successfully!', 'success');
Utils.showToast('Invalid data', 'error');
Utils.showToast('Please review', 'warning');
Utils.showToast('Loading...', 'info');
```

**Features:**
- Auto-dismisses after 4 seconds
- Slide-in/out animations
- Color-coded by type (green, red, orange, blue)
- Stack multiple toasts
- Manual dismiss option

### Confirmation Dialog System

**Usage:**
```javascript
const confirmed = await Utils.confirm('Delete Task', 'Are you sure?');
if (confirmed) {
    // Perform destructive action
}
```

**Features:**
- Promise-based async/await support
- Prevents accidental deletions
- Custom title and message
- Cancel and Confirm buttons

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
- `addAbleTask()` - Creates ABLE task from form with validation
- `addWiseTask()` - Creates WISE task from form with validation
- Both functions validate input, show toast notifications, and call autoSave()

**Editing Tasks:**
- `editTask(taskId, taskOrg)` - Opens edit modal with task data
- `saveTaskEdit()` - Validates and saves changes from edit modal
- Uses professional modal interface (not browser prompts)
- Shows success/error toasts

**Deleting Tasks:**
- `deleteTask(taskId, taskOrg)` - Async function with confirmation dialog
- Shows "Are you sure?" confirmation before deletion
- Removes task from array, calls autoSave(), refreshes all views
- Shows success toast with deleted task name

**Duplicating Tasks:**
- `duplicateTask(taskId, taskOrg)` - Creates copy of existing task
- Appends " (Copy)" to title
- Generates new unique ID
- Shows success toast

**Utility Functions:**
- `refreshAllViews(organization)` - Re-renders all affected views
- `sortMasterTable(column)` - Toggles sort order
- Filters applied in render functions based on input values

**Important:** All task operations call `autoSave()` to persist changes immediately.

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

**Current State:** ✅ FULL LOCAL PERSISTENCE
- All data persists via localStorage
- Automatic save on every data mutation
- Automatic load on page load
- Periodic backup every 30 seconds
- Storage key: `ableWiseDashboardData`

**Storage Implementation:**
- `Storage.save()` - Saves all app data to localStorage
- `Storage.load()` - Loads data on app initialization
- `Storage.clear()` - Clears all saved data
- `autoSave()` - Called after every data change
- Toast notifications inform users of save/load status

**What's Saved:**
- All tasks (main, ABLE, WISE)
- All workflows (ABLE, WISE)
- WISE quick notes
- Sort preferences

### Known Limitations

1. **No backend integration** - purely client-side (data only on this browser)
2. **No user authentication** - admin icon is decorative
3. **No real workflow execution** - workflows saved but not auto-executed
4. **No multi-user collaboration** - single-user application
5. **No data sync across devices** - localStorage is per-browser
6. **No export/import** - cannot backup to file or transfer to other browsers

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

**NEW - Data Persistence:**
- [ ] Data persists after page reload
- [ ] Toast shows on successful load
- [ ] Data saves automatically after changes
- [ ] Periodic backup works (check console every 30s)

**NEW - Task Editing:**
- [ ] Edit button opens modal with current task data
- [ ] Can modify all task fields in modal
- [ ] Save button updates task
- [ ] Toast shows success message
- [ ] All views refresh after edit

**NEW - Task Deletion:**
- [ ] Delete button shows confirmation dialog
- [ ] Can cancel deletion
- [ ] Confirming removes task
- [ ] Toast shows deleted task name
- [ ] All views refresh after deletion

**NEW - Task Duplication:**
- [ ] Copy button duplicates task
- [ ] New task has " (Copy)" appended to title
- [ ] Toast shows success
- [ ] Duplicated task appears in all views

**NEW - Validation & Errors:**
- [ ] Empty title shows error toast
- [ ] Title over 100 chars shows error
- [ ] Invalid date shows error
- [ ] Error toasts are red

**NEW - Toast Notifications:**
- [ ] Success toasts are green
- [ ] Error toasts are red
- [ ] Toasts auto-dismiss after 4 seconds
- [ ] Multiple toasts stack properly
- [ ] Can manually close toasts

**NEW - Mobile Responsiveness:**
- [ ] Navbar adapts on small screens
- [ ] Daily planner stacks vertically on mobile
- [ ] Task table scrolls horizontally if needed
- [ ] Toasts fit on small screens
- [ ] Modals are readable on mobile

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
**Version:** 2.0
**Maintained for:** AI Assistants (Claude Code)

## Version History

### v2.0 (2025-11-16)
- Added localStorage persistence with auto-save
- Implemented task deletion with confirmation
- Added task duplication feature
- Replaced prompts with professional edit modal
- Added toast notification system
- Added confirmation dialog system
- Implemented input validation
- Enhanced mobile responsiveness
- Added comprehensive error handling

### v1.0 (2025-11-16)
- Initial release with core functionality
- Three-view dashboard (Main, ABLE, WISE)
- Task creation and drag-and-drop
- Kanban boards for both organizations
- Calendar and timeline views
- Basic workflow builders
