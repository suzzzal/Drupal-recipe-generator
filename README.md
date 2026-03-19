## 1. User Prompt

### Overview

The **User Prompt** step is the entry point of the system, where users describe the desired Drupal feature in natural language. This input is later transformed into a structured prompt for AI-based recipe generation.

To ensure clarity and usability, this step is implemented using a guided interface built with Drupal Form API.

---

### User Interface Design

The interface is designed to be simple, intuitive, and helpful for both technical and non-technical users.

#### 🔹 Prompt Input Area
- A large textarea where users can describe their requirements in natural language  
- Includes placeholder text (e.g., *“Create a Team Member content type with photo and bio fields…”*)  
- Supports multi-line input for detailed descriptions  

#### 🔹 Feature Hints
- Displays commonly used Drupal concepts to guide users  
- Includes:
  - Content Types  
  - Fields (text, image, date, etc.)  
  - Views (grid, list, table)  
  - Taxonomy (tags, categories)  
  - Media  

This helps users structure their prompts more effectively.

#### 🔹 Example Prompts
- Provides clickable sample prompts such as:
  - “Create a blog content type with title and body”  
  - “Create an event system with date and calendar view”  
- Clicking an example auto-fills the input field  

#### 🔹 Action Button
- A clear call-to-action button: **“Generate Recipe”**  
- Submits the prompt and triggers the AI generation pipeline  

#### 🔹 Optional Enhancements
- Autocomplete suggestions while typing  
- Feature chips (clickable keywords like *Field*, *View*, *Image*)  
- Basic validation for empty or unclear input  

---

### Flow

User opens Recipe Generator UI
↓
Enters prompt (natural language)
↓
(Optional) Uses hints/examples
↓
Submits request


---

### UX Goals

- Guide users to write structured and meaningful prompts  
- Reduce ambiguity in natural language input  
- Improve the quality and accuracy of AI-generated recipes  
- Provide a smooth and beginner-friendly experience  

---

### Suggested Visualization

| Describe your feature |
| [________________________________] |
| |
| Hints: Content Type | Field | View |
| |
| Examples: |
| - Blog system |
| - Event calendar |
| |
| [ Generate Recipe ] |


