---
title: "first post"
description: "this is a test post"
publishDate: "22 Feb 2023"
updatedDate: 22 Jan 2024
tags: ["test", "markdown"]
---

# This is a test

# Prompt-PWA-spec

---

# Developer Specification: Prompt Storage Dashboard PWA

## 1. Project Overview
This project is a Progressive Web App (PWA) designed to store and manage prompts for language models. The app features a robust prompt management system with options to categorize, search, filter, sort, edit, and soft-delete prompts. The UI leverages ShadCN components and is built with a dark mode aesthetic. Cloud storage is provided via Supabase on a free tier, and no user authentication is required at launch.

---

## 2. Functional Requirements

### Core Features
- **Prompt Categorization:**
  - **Folders:** Each prompt is assigned to a single folder.
  - **Tags:** Prompts can have multiple tags that are global (i.e., tags are not restricted to a specific folder).
  
- **Searching & Filtering:**
  - **Text Search:** Ability to search prompts by keywords.
  - **Folder Filtering:** Filter prompts by their assigned folder.
  - **Tag Filtering:** Filter prompts by associated tags.
  - **Sorting Options:** 
    - Sort by date created (newest/oldest).
    - Sort by last modified (recently edited first).
    - Sort alphabetically (A–Z, Z–A).

- **Editing & Deleting:**
  - **CRUD Operations:** Ability to create, edit, and delete prompts.
  - **Soft Deletion:** Instead of immediate removal, deleted prompts are flagged (using a `deleted_at` timestamp) and moved to a “Trash” section, where they can be restored or permanently deleted via manual cleanup.

### Future Enhancements
- **Analytics:**
  - Track usage patterns (e.g., most used prompts, frequency of edits, activity logs).
- **Export/Import Functionality:**
  - Export prompts (e.g., JSON or CSV file format).
  - Import prompts from a file for quick data migration or backup.

---

## 3. Architecture & Technology Choices

### Frontend
- **Framework:** Build as a Progressive Web App (PWA) using a modern JavaScript framework such as React (Next.js is a good candidate due to its built-in PWA support and server-side rendering capabilities).
- **UI Components:** Utilize ShadCN’s component library to create:
  - **Buttons:** For actions such as “Add Prompt,” “Delete,” “Restore.”
  - **Modals:** For editing prompts and confirming delete actions.
  - **Cards/List Items:** For presenting prompts in a clear, card-based or list-based layout.
  - **Alerts/Notifications:** To provide user feedback on operations (e.g., "Prompt deleted successfully").
- **Layout:**
  - **Sidebar:** A collapsible sidebar dedicated to folder and tag filtering.
  - **Top Search Bar:** Incorporates text search and dropdown menus for sorting.
  - **Visual Style:** Dark mode to ensure a modern, low-light friendly interface.

### Backend & Cloud Storage
- **Database & API:** Use Supabase’s free tier which provides:
  - A PostgreSQL database.
  - A RESTful API for database interactions.
  - Built-in support for real-time subscriptions if needed.
- **Data Sync:** All CRUD operations and search/filter queries will be executed via Supabase’s API endpoints.

---

## 4. Data Model & Database Schema

### Tables & Fields

1. **Prompts Table**
   - **id:** UUID (primary key)
   - **title:** Text (short description or title for the prompt)
   - **content:** Text (the full prompt text)
   - **folder_id:** UUID (foreign key linking to the Folders table)
   - **created_at:** Timestamp (auto-generated)
   - **updated_at:** Timestamp (auto-updated on edit)
   - **deleted_at:** Timestamp (nullable; set when a prompt is soft-deleted)

2. **Folders Table**
   - **id:** UUID (primary key)
   - **name:** Text (folder name)
   - **created_at:** Timestamp

3. **Tags Table**
   - **id:** UUID (primary key)
   - **name:** Text (tag name)
   - **created_at:** Timestamp

4. **Prompt_Tags (Join Table)**
   - **prompt_id:** UUID (foreign key to Prompts)
   - **tag_id:** UUID (foreign key to Tags)
   - **Primary Key:** Composite key of (prompt_id, tag_id)

### Data Handling Notes
- **Soft Deletion:**  
  Instead of a hard delete, setting `deleted_at` to the current timestamp flags a prompt as deleted. Queries for active prompts must filter out records where `deleted_at` is not null.
- **Queries:**
  - Fetch active prompts: `WHERE deleted_at IS NULL`
  - Fetch trash prompts: `WHERE deleted_at IS NOT NULL`

---

## 5. UI/UX Requirements

### Layout & Navigation
- **Sidebar (Collapsible):**
  - Lists available **Folders**.
  - Lists global **Tags**.
  - Clicking an item applies the respective filter.
  
- **Top Navigation Bar:**
  - **Search Bar:** For free-text search across prompts.
  - **Sorting Dropdowns:** Allow selection of sort criteria (date created, last modified, alphabetical).
  
- **Main Content Area:**
  - Displays prompts using cards or list items.
  - Provides action buttons (edit, soft-delete, restore) on each prompt.
  - Includes a “Trash” section accessible via a dedicated tab or link for managing soft-deleted items.

### ShadCN Components Usage
- **Buttons:** Consistent styling for all actions.
- **Modals:** For prompt editing and delete confirmation dialogues.
- **Cards/List Items:** For clean, accessible display of prompt information.
- **Alerts/Notifications:** To notify users of success or failure of operations.
- **Extensibility:** Design components with scalability in mind, allowing the addition of new components or features later.

---

## 6. Error Handling Strategies

### Frontend Error Handling
- **API Requests:**
  - Use `try/catch` around all asynchronous API calls.
  - Display descriptive error alerts using ShadCN’s alert component if an API call fails.
- **Form Validation:**
  - Validate prompt input fields (e.g., ensure title and content are not empty) before submission.
  - Provide inline feedback for invalid inputs.
  
### Backend & Data Layer
- **Soft Deletion Consistency:**
  - Ensure that every “delete” operation only sets the `deleted_at` field rather than removing the record.
- **API Response Checks:**
  - Check for non-200 HTTP status codes from Supabase and log errors.
  - Implement retry logic for transient network errors if needed.
  
### Logging & Monitoring
- Consider integrating basic client-side logging (or Supabase logging features) to capture errors for debugging purposes.

---

## 7. Testing Plan

### Unit Testing
- **Components:**
  - Test each UI component (buttons, modals, cards, alerts) using Jest and React Testing Library.
  - Verify component rendering, user interaction (e.g., clicking a button opens a modal), and state changes.
  
### Integration Testing
- **API Interactions:**
  - Mock Supabase API responses to test CRUD operations.
  - Ensure that soft deletion, restoration, and filtering logic work as expected.
  
### End-to-End (E2E) Testing
- **User Flows:**
  - Use Cypress or a similar tool to test:
    - Creating a new prompt.
    - Editing an existing prompt.
    - Soft-deleting a prompt and verifying it appears in the Trash section.
    - Restoring a prompt from Trash.
    - Searching and filtering by folder and tag.
- **Offline & PWA Testing:**
  - Test PWA capabilities including service worker registration, offline access, and caching strategies.

### Continuous Integration (CI)
- Set up a CI pipeline (using GitHub Actions, GitLab CI, etc.) to run tests on every commit to maintain code quality.

---

## 8. Deployment & Additional Considerations

### PWA Specifics
- **Service Worker:**  
  - Implement a service worker to handle caching of assets and API responses.
- **Manifest File:**
  - Configure the web app manifest to ensure the app is installable on devices.
- **Responsive Design:**  
  - Ensure the dashboard is responsive and works on various devices (desktop, tablet, mobile).

### Future Enhancements Roadmap
- **Analytics Module:**  
  - Integrate an analytics dashboard to track prompt usage and editing frequency.
- **Export/Import Module:**  
  - Provide options to export prompts (as JSON/CSV) and import from files.
- **User Authentication (Planned):**  
  - Although not required initially, structure the code to allow future integration of Supabase authentication for personalized prompt management.

---

By following this detailed specification, the developer will have clear guidance on both the functional and technical requirements needed to build and extend the Prompt Storage Dashboard PWA. This document covers architecture choices, data handling practices, error handling strategies, and a robust testing plan to ensure a high-quality, scalable application.