# Product Requirements Document (PRD) - TODO App Enhancement: Due Dates, Priorities & Filters

## 1. Overview

We are upgrading the basic TODO app to support due dates, priorities, and filters so users can better organize and prioritize their tasks. The current app only supports task title and completion status. This enhancement will enable users to set deadlines, assign priority levels, and filter tasks by relevance (All, Today, Overdue), making the app more practical for real-world task management without adding unnecessary complexity.

The goal is to deliver a simple, teachable MVP that significantly improves task organization while maintaining the app's ease of use. Storage will remain local-only with no backend changes required.

---

## 2. MVP Scope

### Core Features
- **Due Date Field**
  - Add optional `dueDate` field to each task
  - Format: ISO date string (`YYYY-MM-DD`)
  - Invalid date values should be ignored and treated as absent
  - Tasks without due dates are permitted

- **Priority Field**
  - Add `priority` field to each task
  - Values: `"P1"`, `"P2"`, or `"P3"`
  - Default value: `"P3"` (lowest priority)
  - Field is required with default applied

- **Filter Tabs**
  - Implement three filter views:
    - **All**: Display all tasks (including completed tasks)
    - **Today**: Display only incomplete tasks with due date matching today's date
    - **Overdue**: Display only incomplete tasks with due date before today's date
  - Filters are mutually exclusive (user can view one filter at a time)

### Data Model Requirements
- `title`: string, required (existing field)
- `completed`: boolean (existing field)
- `priority`: enum `"P1" | "P2" | "P3"`, default `"P3"`
- `dueDate`: optional string in ISO `YYYY-MM-DD` format

### Storage
- Continue using local storage (no backend or external storage changes)
- Maintain existing localStorage implementation

### Validation
- Task title remains required
- Priority must be one of the three valid values
- Due date, if provided, should be validated as ISO `YYYY-MM-DD` format
- Invalid due dates should be gracefully handled (ignored/treated as absent)

---

## 3. Post-MVP Scope

### Visual Enhancements
- **Overdue Task Highlighting**
  - Display overdue tasks with visual prominence (e.g., red highlighting or border)
  - Make it immediately obvious which tasks are past due

- **Priority Color Badges**
  - Display color-coded visual indicators for priority levels:
    - P1: Red badge/indicator
    - P2: Orange badge/indicator
    - P3: Gray badge/indicator

### Advanced Sorting
- **Multi-Level Sort Logic**
  - Primary: Overdue tasks appear first
  - Secondary: Sort by priority (P1 → P2 → P3)
  - Tertiary: Sort by due date (ascending/earliest first)
  - Quaternary: Tasks without due dates appear last
  - Maintain stable sort for tasks with identical sorting criteria

---

## 4. Out of Scope

- **Notifications**: No email, push, or in-app notifications for approaching or overdue tasks
- **Recurring Tasks**: No support for repeating tasks or task templates
- **Multi-User Functionality**: No collaboration, sharing, or multi-user access features
- **Keyboard Navigation**: No special keyboard shortcuts or accessibility enhancements
- **External Storage**: No backend database, cloud sync, or external storage integration
- **Advanced Accessibility Features**: Beyond basic HTML semantics
- **Task Categories/Tags**: No additional organizational metadata beyond priority
- **Task Notes/Descriptions**: No extended text fields beyond the title
- **Task Dependencies**: No relationships or dependencies between tasks
- **Time-of-Day Tracking**: Due dates are day-level only, not specific times

---

## Document History

| Date | Version | Author | Changes |
|------|---------|--------|---------|
| Sept 16, 2025 | 0.1 | Consultant | Initial draft from requirements meeting |
| Sept 17, 2025 | 1.0 | Consultant | Finalized MVP/Post-MVP split via Slack |
