# 📘 HTTP Methods and FastAPI Project Notes

## 🌐 Understanding HTTP Methods

### Software Categories Based on User Interaction
- **Static Software**  
  - Minimal user interaction.  
  - Primarily used for receiving information.  
  - Examples: Calendar, Clock.  

- **Dynamic Software**  
  - High user interaction with two-way communication.  
  - Examples: Microsoft Excel, Word, PowerPoint.  

---

### CRUD Operations in Dynamic Software
Dynamic software interactions can be categorized into **CRUD**:
- **C - Create**: Add new data (e.g., new Excel cell, new Word document).  
- **R - Retrieve**: View or read existing data (e.g., reading a Word document).  
- **U - Update**: Modify existing data (e.g., editing Excel cells).  
- **D - Delete**: Remove existing data (e.g., deleting a Word document).  

---

### Websites and HTTP Protocol
- Websites are applications installed on a **server** and accessed by **clients**.  
- Communication happens via the **HTTP protocol**.  

#### Types of Websites
- **Static Websites**: Minimal interaction (e.g., blogs, government sites).  
- **Dynamic Websites**: High interaction (e.g., Instagram, Zomato).  
  - All interactions still map to CRUD operations.  

---

### HTTP Verbs/Methods for CRUD
- **GET**: Retrieve information (e.g., requesting a profile page).  
- **POST**: Create new resources or send data (e.g., login/registration form).  
- **PUT**: Update an existing resource.  
- **DELETE**: Remove a resource.  

> These verbs are collectively called **HTTP Methods**.  
> - **GET** and **POST** are most common.  
> - **PUT** and **DELETE** are less frequent.  
