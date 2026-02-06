# Epics and Stories - TODO App Enhancement

## MVP

- Epic: Due Date Management
  - Story: Add due date field to task form
    - AC: Due date input field is present in task form
    - AC: Due date field accepts ISO YYYY-MM-DD format
    - AC: Due date field is optional (tasks can be created without a due date)
    - AC: Due date input uses appropriate HTML input type for date selection
    - Tech: Already implemented - TextField with type="date" exists in TaskForm.js
    - Tech: Verify date normalization in normalizeDateString() function handles edge cases
    - Tech: Ensure due_date state variable properly integrates with form submission
  
  - Story: Store due date in ISO YYYY-MM-DD format
    - AC: Due date is stored as ISO YYYY-MM-DD string in localStorage
    - AC: Stored due date persists across page reloads
    - AC: Due date can be retrieved and displayed correctly after storage
    - Tech: Already implemented - Backend app.js stores due_date as DATE type in SQLite
    - Tech: Backend accepts due_date in POST /api/tasks and PUT /api/tasks/:id
    - Tech: Frontend onSave() sends due_date: dueDate in request body
  
  - Story: Handle invalid due date values gracefully
    - AC: Invalid date formats are ignored and treated as absent
    - AC: Tasks with invalid dates are saved without due dates
    - AC: Application does not crash or display errors for invalid dates
    - AC: User can continue using the app when invalid date is entered
    - Tech: Add validation in backend app.js POST/PUT endpoints to check date format
    - Tech: Use regex /^\d{4}-\d{2}-\d{2}$/ or Date.parse() to validate
    - Tech: Store null if validation fails, converting due_date || null in SQL statement
    - Tech: Frontend normalizeDateString() should handle invalid inputs by returning empty string

- Epic: Task Prioritization
  - Story: Add priority field to task form
    - AC: Priority selector is present in task form
    - AC: Priority options P1, P2, and P3 are available for selection
    - AC: Priority selector is user-friendly (dropdown or radio buttons)
    - Tech: Add priority state in TaskForm.js: const [priority, setPriority] = useState('P3')
    - Tech: Add MUI Select or RadioGroup component with options: P1, P2, P3
    - Tech: Include priority in onSave payload: await onSave({ title, description, due_date: dueDate, priority })
    - Tech: Update useEffect to set priority from initialTask?.priority || 'P3'
  
  - Story: Set default priority to P3
    - AC: New tasks default to P3 if no priority is explicitly selected
    - AC: P3 is pre-selected in priority selector for new tasks
    - AC: Default priority is applied before task is saved
    - Tech: Initialize priority state with 'P3': useState('P3')
    - Tech: In backend app.js, add default in CREATE TABLE: priority TEXT DEFAULT 'P3'
    - Tech: Backend POST endpoint should use: priority || 'P3' when inserting
  
  - Story: Validate priority values (P1, P2, P3)
    - AC: Only P1, P2, and P3 values are accepted and stored
    - AC: Invalid priority values are rejected or converted to default
    - AC: Stored priority value is one of the three valid options
    - Tech: Add validation in backend POST/PUT endpoints: if (!['P1', 'P2', 'P3'].includes(priority)) priority = 'P3'
    - Tech: Frontend Select component should restrict options to valid values
    - Tech: Add CHECK constraint in SQLite: priority TEXT CHECK(priority IN ('P1', 'P2', 'P3')) DEFAULT 'P3'

- Epic: Task Filtering
  - Story: Add filter tabs (All, Today, Overdue)
    - AC: Three filter tabs are visible (All, Today, Overdue)
    - AC: User can click each tab to switch between filters
    - AC: Active filter tab is visually indicated
    - Tech: Add filter state in TaskList.js or App.js: const [activeFilter, setActiveFilter] = useState('all')
    - Tech: Use MUI Tabs component with three Tab elements: 'all', 'today', 'overdue'
    - Tech: Style active tab using MUI sx prop or Tabs indicator
    - Tech: Position tabs above task list in TaskList.js Paper component
  
  - Story: Implement All filter to show all tasks
    - AC: All filter shows both complete and incomplete tasks
    - AC: All tasks are visible regardless of due date or priority
    - AC: Completed tasks appear in All filter view
    - Tech: When activeFilter === 'all', display all tasks without filtering
    - Tech: Remove completed query param from GET /api/tasks when All is selected
    - Tech: Current fetchTasks() implementation already fetches all by default
  
  - Story: Implement Today filter for incomplete tasks due today
    - AC: Today filter shows only incomplete tasks
    - AC: Only tasks with due date matching current date are shown
    - AC: Completed tasks are excluded from Today filter
    - AC: Tasks without due dates are not shown in Today filter
    - Tech: Add client-side filtering in TaskList.js after fetching tasks
    - Tech: Get today's date: const today = new Date().toISOString().split('T')[0]
    - Tech: Filter logic: tasks.filter(t => !t.completed && t.due_date === today)
    - Tech: Alternative: Add query param to backend GET /api/tasks?filter=today and handle server-side
  
  - Story: Implement Overdue filter for incomplete tasks past due
    - AC: Overdue filter shows only incomplete tasks
    - AC: Only tasks with due date before current date are shown
    - AC: Completed tasks are excluded from Overdue filter
    - AC: Tasks without due dates are not shown in Overdue filter
    - Tech: Add client-side filtering in TaskList.js after fetching tasks
    - Tech: Get today's date: const today = new Date().toISOString().split('T')[0]
    - Tech: Filter logic: tasks.filter(t => !t.completed && t.due_date && t.due_date < today)
    - Tech: Alternative: Add query param to backend GET /api/tasks?filter=overdue and handle server-side
  
  - Story: Make filters mutually exclusive
    - AC: Only one filter can be active at a time
    - AC: Selecting a filter deselects other filters automatically
    - AC: Filter selection updates task list immediately
    - Tech: Use single activeFilter state variable (string: 'all', 'today', 'overdue')
    - Tech: MUI Tabs component naturally enforces single selection
    - Tech: Update displayed tasks when activeFilter changes via useEffect or direct computation
    - Tech: Apply filter in render based on activeFilter value before mapping tasks

- Epic: Data Model Enhancement
  - Story: Update task data model with priority and dueDate fields
    - AC: Task objects include priority field (P1, P2, or P3)
    - AC: Task objects include dueDate field (optional ISO YYYY-MM-DD string)
    - AC: Existing title and completed fields are preserved
    - AC: Existing tasks are migrated to include new fields with defaults
    - Tech: Modify db.exec() in backend app.js CREATE TABLE to add priority column
    - Tech: Add: priority TEXT CHECK(priority IN ('P1', 'P2', 'P3')) DEFAULT 'P3'
    - Tech: due_date DATE column already exists in current schema
    - Tech: Update POST and PUT endpoints to accept priority field
    - Tech: Migration: ALTER TABLE tasks ADD COLUMN priority TEXT DEFAULT 'P3' (if needed for existing data)
  
  - Story: Maintain localStorage implementation
    - AC: Tasks persist in localStorage after creation/update
    - AC: No external or backend storage is used
    - AC: localStorage implementation remains unchanged in approach
    - Tech: NOTE: Current implementation uses SQLite in-memory database via backend
    - Tech: For true localStorage: Add browser localStorage.setItem/getItem in frontend
    - Tech: Store tasks array as JSON: localStorage.setItem('tasks', JSON.stringify(tasks))
    - Tech: Load on mount: JSON.parse(localStorage.getItem('tasks') || '[]')
    - Tech: Remove backend API calls if switching to localStorage-only approach
  
  - Story: Validate task title as required field
    - AC: Tasks cannot be created without a title
    - AC: Empty or whitespace-only title values are rejected
    - AC: User receives indication that title is required
    - Tech: Already implemented - TaskForm.js checks: if (!title.trim()) { setError('Title is required'); return; }
    - Tech: Backend validates: if (!title || typeof title !== 'string' || title.trim() === '') return 400 error
    - Tech: TextField has required prop in TaskForm.js
    - Tech: Error message displayed via error state variable in Typography component

## Post-MVP

- Epic: Visual Task Indicators
  - Story: Highlight overdue tasks in red
    - AC: Overdue tasks display with red highlighting or red border
    - AC: Visual prominence makes overdue status immediately obvious
    - AC: Red highlighting is applied consistently across all views
    - Tech: Add isOverdue helper function in TaskList.js to check if task.due_date < today
    - Tech: Modify ListItem sx prop based on overdue status
    - Tech: Apply conditional styling: borderColor: isOverdue ? '#f44336' : existing values
    - Tech: Add background: isOverdue ? 'rgba(244, 67, 54, 0.08)' : existing values
    - Tech: Consider using Box wrapper with red left border for stronger visual indicator
  
  - Story: Add red priority badge for P1 tasks
    - AC: P1 tasks display red color badge or indicator
    - AC: Badge is clearly visible on task item
    - AC: Badge color distinguishes P1 from other priorities
    - Tech: Use MUI Chip component similar to existing due date chip in TaskList.js
    - Tech: Add condition: {task.priority === 'P1' && <Chip label="P1" size="small" sx={{...}} />}
    - Tech: Style with: background: '#f44336' (red), color: 'white', fontWeight: 600
    - Tech: Position near existing due date Chip in absolute positioned Box
  
  - Story: Add orange priority badge for P2 tasks
    - AC: P2 tasks display orange color badge or indicator
    - AC: Badge is clearly visible on task item
    - AC: Badge color distinguishes P2 from other priorities
    - Tech: Use MUI Chip component similar to P1 badge
    - Tech: Add condition: {task.priority === 'P2' && <Chip label="P2" size="small" sx={{...}} />}
    - Tech: Style with: background: '#ff9800' (orange), color: 'white', fontWeight: 600
    - Tech: Position consistently with P1 badge
  
  - Story: Add gray priority badge for P3 tasks
    - AC: P3 tasks display gray color badge or indicator
    - AC: Badge is clearly visible on task item
    - AC: Badge color distinguishes P3 from other priorities
    - Tech: Use MUI Chip component similar to P1 and P2 badges
    - Tech: Add condition: {task.priority === 'P3' && <Chip label="P3" size="small" sx={{...}} />}
    - Tech: Style with: background: '#9e9e9e' (gray), color: 'white', fontWeight: 600
    - Tech: Position consistently with other priority badges
    - Tech: Consider making P3 optional or more subtle since it's default priority

- Epic: Advanced Task Sorting
  - Story: Sort overdue tasks first
    - AC: Overdue tasks appear at top of task list
    - AC: Overdue tasks are grouped together before all other tasks
    - AC: Sort order updates when tasks become overdue
    - Tech: Create custom sort function in TaskList.js before mapping tasks
    - Tech: Add helper: const isOverdue = (task) => task.due_date && task.due_date < today
    - Tech: Sort logic: tasks.sort((a, b) => { if (isOverdue(a) && !isOverdue(b)) return -1; ... })
    - Tech: Alternative: Implement in backend GET /api/tasks with ORDER BY clause
  
  - Story: Sort tasks by priority level (P1, P2, P3)
    - AC: Within same overdue status, P1 tasks appear before P2 tasks
    - AC: P2 tasks appear before P3 tasks within same group
    - AC: Priority sorting is consistent across all filter views
    - Tech: Extend sort function to include priority after overdue check
    - Tech: Priority order map: const priorityOrder = { P1: 1, P2: 2, P3: 3 }
    - Tech: Sort logic: if (isOverdue(a) === isOverdue(b)) return priorityOrder[a.priority] - priorityOrder[b.priority]
    - Tech: Ensure priority field exists on all tasks (use default 'P3' if missing)
  
  - Story: Sort tasks by due date ascending
    - AC: Within same priority level, earlier due dates appear first
    - AC: Due dates are sorted in ascending chronological order
    - AC: Sort handles tasks with same due date gracefully
    - Tech: Extend sort function to include due date after priority check
    - Tech: Handle null due dates: const dueDateA = a.due_date || '9999-12-31'
    - Tech: Sort logic: if (priorityOrder[a.priority] === priorityOrder[b.priority]) return dueDateA.localeCompare(dueDateB)
    - Tech: ISO date strings naturally sort correctly with string comparison
  
  - Story: Place tasks without due dates last
    - AC: Tasks without due dates appear at bottom of list
    - AC: Undated tasks come after all tasks with due dates
    - AC: Undated tasks are still sorted by priority among themselves
    - Tech: Handle null due_date in sort function by using high sentinel value
    - Tech: Undated tasks sorting: const dueDateA = a.due_date || '9999-12-31'
    - Tech: This ensures undated tasks sort to end while maintaining priority order
    - Tech: Current backend ORDER BY due_date IS NULL, due_date ASC already implements this pattern
    - Tech: Frontend can replicate or rely on backend ordering
