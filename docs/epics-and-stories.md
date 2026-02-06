# Epics and Stories - TODO App Enhancement

## MVP

### Epic: Due Date Management

#### Story: Add due date field to task form
**Acceptance Criteria:**
- Due date input field is present in task form
- Due date field accepts ISO YYYY-MM-DD format
- Due date field is optional (tasks can be created without a due date)
- Due date input uses appropriate HTML input type for date selection

**Technical Requirements:**
- Already implemented - TextField with type="date" exists in TaskForm.js
- Verify date normalization in normalizeDateString() function handles edge cases
- Ensure due_date state variable properly integrates with form submission

#### Story: Store due date in ISO YYYY-MM-DD format
**Acceptance Criteria:**
- Due date is stored as ISO YYYY-MM-DD string in localStorage
- Stored due date persists across page reloads
- Due date can be retrieved and displayed correctly after storage

**Technical Requirements:**
- Already implemented - Backend app.js stores due_date as DATE type in SQLite
- Backend accepts due_date in POST /api/tasks and PUT /api/tasks/:id
- Frontend onSave() sends due_date: dueDate in request body

#### Story: Handle invalid due date values gracefully
**Acceptance Criteria:**
- Invalid date formats are ignored and treated as absent
- Tasks with invalid dates are saved without due dates
- Application does not crash or display errors for invalid dates
- User can continue using the app when invalid date is entered

**Technical Requirements:**
- Add validation in backend app.js POST/PUT endpoints to check date format
- Use regex /^\d{4}-\d{2}-\d{2}$/ or Date.parse() to validate
- Store null if validation fails, converting due_date || null in SQL statement
- Frontend normalizeDateString() should handle invalid inputs by returning empty string

---

### Epic: Task Prioritization

#### Story: Add priority field to task form
**Acceptance Criteria:**
- Priority selector is present in task form
- Priority options P1, P2, and P3 are available for selection
- Priority selector is user-friendly (dropdown or radio buttons)

**Technical Requirements:**
- Add priority state in TaskForm.js: const [priority, setPriority] = useState('P3')
- Add MUI Select or RadioGroup component with options: P1, P2, P3
- Include priority in onSave payload: await onSave({ title, description, due_date: dueDate, priority })
- Update useEffect to set priority from initialTask?.priority || 'P3'

#### Story: Set default priority to P3
**Acceptance Criteria:**
- New tasks default to P3 if no priority is explicitly selected
- P3 is pre-selected in priority selector for new tasks
- Default priority is applied before task is saved

**Technical Requirements:**
- Initialize priority state with 'P3': useState('P3')
- In backend app.js, add default in CREATE TABLE: priority TEXT DEFAULT 'P3'
- Backend POST endpoint should use: priority || 'P3' when inserting

#### Story: Validate priority values (P1, P2, P3)
**Acceptance Criteria:**
- Only P1, P2, and P3 values are accepted and stored
- Invalid priority values are rejected or converted to default
- Stored priority value is one of the three valid options

**Technical Requirements:**
- Add validation in backend POST/PUT endpoints: if (!['P1', 'P2', 'P3'].includes(priority)) priority = 'P3'
- Frontend Select component should restrict options to valid values
- Add CHECK constraint in SQLite: priority TEXT CHECK(priority IN ('P1', 'P2', 'P3')) DEFAULT 'P3'

---

### Epic: Task Filtering

#### Story: Add filter tabs (All, Today, Overdue)
**Acceptance Criteria:**
- Three filter tabs are visible (All, Today, Overdue)
- User can click each tab to switch between filters
- Active filter tab is visually indicated

**Technical Requirements:**
- Add filter state in TaskList.js or App.js: const [activeFilter, setActiveFilter] = useState('all')
- Use MUI Tabs component with three Tab elements: 'all', 'today', 'overdue'
- Style active tab using MUI sx prop or Tabs indicator
- Position tabs above task list in TaskList.js Paper component

#### Story: Implement All filter to show all tasks
**Acceptance Criteria:**
- All filter shows both complete and incomplete tasks
- All tasks are visible regardless of due date or priority
- Completed tasks appear in All filter view

**Technical Requirements:**
- When activeFilter === 'all', display all tasks without filtering
- Remove completed query param from GET /api/tasks when All is selected
- Current fetchTasks() implementation already fetches all by default

#### Story: Implement Today filter for incomplete tasks due today
**Acceptance Criteria:**
- Today filter shows only incomplete tasks
- Only tasks with due date matching current date are shown
- Completed tasks are excluded from Today filter
- Tasks without due dates are not shown in Today filter

**Technical Requirements:**
- Add client-side filtering in TaskList.js after fetching tasks
- Get today's date: const today = new Date().toISOString().split('T')[0]
- Filter logic: tasks.filter(t => !t.completed && t.due_date === today)
- Alternative: Add query param to backend GET /api/tasks?filter=today and handle server-side

#### Story: Implement Overdue filter for incomplete tasks past due
**Acceptance Criteria:**
- Overdue filter shows only incomplete tasks
- Only tasks with due date before current date are shown
- Completed tasks are excluded from Overdue filter
- Tasks without due dates are not shown in Overdue filter

**Technical Requirements:**
- Add client-side filtering in TaskList.js after fetching tasks
- Get today's date: const today = new Date().toISOString().split('T')[0]
- Filter logic: tasks.filter(t => !t.completed && t.due_date && t.due_date < today)
- Alternative: Add query param to backend GET /api/tasks?filter=overdue and handle server-side

#### Story: Make filters mutually exclusive
**Acceptance Criteria:**
- Only one filter can be active at a time
- Selecting a filter deselects other filters automatically
- Filter selection updates task list immediately

**Technical Requirements:**
- Use single activeFilter state variable (string: 'all', 'today', 'overdue')
- MUI Tabs component naturally enforces single selection
- Update displayed tasks when activeFilter changes via useEffect or direct computation
- Apply filter in render based on activeFilter value before mapping tasks

---

### Epic: Data Model Enhancement

#### Story: Update task data model with priority and dueDate fields
**Acceptance Criteria:**
- Task objects include priority field (P1, P2, or P3)
- Task objects include dueDate field (optional ISO YYYY-MM-DD string)
- Existing title and completed fields are preserved
- Existing tasks are migrated to include new fields with defaults

**Technical Requirements:**
- Modify db.exec() in backend app.js CREATE TABLE to add priority column
- Add: priority TEXT CHECK(priority IN ('P1', 'P2', 'P3')) DEFAULT 'P3'
- due_date DATE column already exists in current schema
- Update POST and PUT endpoints to accept priority field
- Migration: ALTER TABLE tasks ADD COLUMN priority TEXT DEFAULT 'P3' (if needed for existing data)

#### Story: Maintain localStorage implementation
**Acceptance Criteria:**
- Tasks persist in localStorage after creation/update
- No external or backend storage is used
- localStorage implementation remains unchanged in approach

**Technical Requirements:**
- NOTE: Current implementation uses SQLite in-memory database via backend
- For true localStorage: Add browser localStorage.setItem/getItem in frontend
- Store tasks array as JSON: localStorage.setItem('tasks', JSON.stringify(tasks))
- Load on mount: JSON.parse(localStorage.getItem('tasks') || '[]')
- Remove backend API calls if switching to localStorage-only approach

#### Story: Validate task title as required field
**Acceptance Criteria:**
- Tasks cannot be created without a title
- Empty or whitespace-only title values are rejected
- User receives indication that title is required

**Technical Requirements:**
- Already implemented - TaskForm.js checks: if (!title.trim()) { setError('Title is required'); return; }
- Backend validates: if (!title || typeof title !== 'string' || title.trim() === '') return 400 error
- TextField has required prop in TaskForm.js
- Error message displayed via error state variable in Typography component

---

## Post-MVP

### Epic: Visual Task Indicators

#### Story: Highlight overdue tasks in red
**Acceptance Criteria:**
- Overdue tasks display with red highlighting or red border
- Visual prominence makes overdue status immediately obvious
- Red highlighting is applied consistently across all views

**Technical Requirements:**
- Add isOverdue helper function in TaskList.js to check if task.due_date < today
- Modify ListItem sx prop based on overdue status
- Apply conditional styling: borderColor: isOverdue ? '#f44336' : existing values
- Add background: isOverdue ? 'rgba(244, 67, 54, 0.08)' : existing values
- Consider using Box wrapper with red left border for stronger visual indicator

#### Story: Add red priority badge for P1 tasks
**Acceptance Criteria:**
- P1 tasks display red color badge or indicator
- Badge is clearly visible on task item
- Badge color distinguishes P1 from other priorities

**Technical Requirements:**
- Use MUI Chip component similar to existing due date chip in TaskList.js
- Add condition: {task.priority === 'P1' && <Chip label="P1" size="small" sx={{...}} />}
- Style with: background: '#f44336' (red), color: 'white', fontWeight: 600
- Position near existing due date Chip in absolute positioned Box

#### Story: Add orange priority badge for P2 tasks
**Acceptance Criteria:**
- P2 tasks display orange color badge or indicator
- Badge is clearly visible on task item
- Badge color distinguishes P2 from other priorities

**Technical Requirements:**
- Use MUI Chip component similar to P1 badge
- Add condition: {task.priority === 'P2' && <Chip label="P2" size="small" sx={{...}} />}
- Style with: background: '#ff9800' (orange), color: 'white', fontWeight: 600
- Position consistently with P1 badge

#### Story: Add gray priority badge for P3 tasks
**Acceptance Criteria:**
- P3 tasks display gray color badge or indicator
- Badge is clearly visible on task item
- Badge color distinguishes P3 from other priorities

**Technical Requirements:**
- Use MUI Chip component similar to P1 and P2 badges
- Add condition: {task.priority === 'P3' && <Chip label="P3" size="small" sx={{...}} />}
- Style with: background: '#9e9e9e' (gray), color: 'white', fontWeight: 600
- Position consistently with other priority badges
- Consider making P3 optional or more subtle since it's default priority

---

### Epic: Advanced Task Sorting

#### Story: Sort overdue tasks first
**Acceptance Criteria:**
- Overdue tasks appear at top of task list
- Overdue tasks are grouped together before all other tasks
- Sort order updates when tasks become overdue

**Technical Requirements:**
- Create custom sort function in TaskList.js before mapping tasks
- Add helper: const isOverdue = (task) => task.due_date && task.due_date < today
- Sort logic: tasks.sort((a, b) => { if (isOverdue(a) && !isOverdue(b)) return -1; ... })
- Alternative: Implement in backend GET /api/tasks with ORDER BY clause

#### Story: Sort tasks by priority level (P1, P2, P3)
**Acceptance Criteria:**
- Within same overdue status, P1 tasks appear before P2 tasks
- P2 tasks appear before P3 tasks within same group
- Priority sorting is consistent across all filter views

**Technical Requirements:**
- Extend sort function to include priority after overdue check
- Priority order map: const priorityOrder = { P1: 1, P2: 2, P3: 3 }
- Sort logic: if (isOverdue(a) === isOverdue(b)) return priorityOrder[a.priority] - priorityOrder[b.priority]
- Ensure priority field exists on all tasks (use default 'P3' if missing)

#### Story: Sort tasks by due date ascending
**Acceptance Criteria:**
- Within same priority level, earlier due dates appear first
- Due dates are sorted in ascending chronological order
- Sort handles tasks with same due date gracefully

**Technical Requirements:**
- Extend sort function to include due date after priority check
- Handle null due dates: const dueDateA = a.due_date || '9999-12-31'
- Sort logic: if (priorityOrder[a.priority] === priorityOrder[b.priority]) return dueDateA.localeCompare(dueDateB)
- ISO date strings naturally sort correctly with string comparison

#### Story: Place tasks without due dates last
**Acceptance Criteria:**
- Tasks without due dates appear at bottom of list
- Undated tasks come after all tasks with due dates
- Undated tasks are still sorted by priority among themselves

**Technical Requirements:**
- Handle null due_date in sort function by using high sentinel value
- Undated tasks sorting: const dueDateA = a.due_date || '9999-12-31'
- This ensures undated tasks sort to end while maintaining priority order
- Current backend ORDER BY due_date IS NULL, due_date ASC already implements this pattern
- Frontend can replicate or rely on backend ordering
