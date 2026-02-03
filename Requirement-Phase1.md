# Badminton SportMatcher – Simple MVP Requirements

## 1. Goal
Build a simple web app that allows badminton players to:
- Create a recruitment post for friendly badminton matches
- Automatically post the content to predefined Facebook Groups
- View the posting result

This MVP focuses only on the **basic happy flow**.

---

## 2. Scope
### In Scope
- Web app
- Create a recruitment post
- Automatically post to Facebook Groups
- View posting result

---

## 3. Basic Flow (Happy Path)

1. User opens the web app  
2. User creates a recruitment post  
3. System automatically posts the content to predefined Facebook Groups  
4. User sees the posting result  

---

## 4. Recruitment Post

### Fields
- Title
- Location
- Date & time
- Skill level (free text)
- Notes (optional)

---

## 5. Facebook Group Posting

### Description
- After submission, the system automatically posts the recruitment content to multiple Facebook Groups

### Requirements
- Facebook Groups are predefined in the system
- Posting uses a central Facebook account
- The same content is posted to all configured groups
- Posted content includes:
  - Recruitment post details
  - Link back to the web app (optional)

---

## 6. View Result

### Description
- User can view the created post and confirm it was posted

### Display
- Recruitment post content
- List of Facebook Group links where the post was published (if available)
- Simple confirmation message, e.g.:
  - “Posted to Facebook Groups”

