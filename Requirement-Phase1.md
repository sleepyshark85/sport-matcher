# Badminton SportMatcher – Simple MVP Requirements

## 1. Goal
- Build a simple MVP web app
---

## 2. Basic Flow (Happy Path)

1. User opens the web app  
2. User creates a recruitment post  
3. System automatically posts the content to predefined Facebook Groups  
4. User sees the posting result  

---

## 3. Recruitment Post

### Fields
- Title
- Location
- Date & time
- Skill level (free text)
- Notes (optional)

---

## 4. Facebook Group Posting

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

## 5. View Result

### Description
- User can view the created post and confirm it was posted

### Display
- Recruitment post content
- List of Facebook Group links where the post was published (if available)
- Simple confirmation message, e.g.:
  - “Posted to Facebook Groups”

