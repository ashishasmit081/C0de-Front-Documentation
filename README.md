# 🚀 C0de Front

## 📚 Table of Contents

1. [Introduction](#1-introduction)
2. [Problem Statement](#2-problem-statement)
3. [Features](#3-features)
4. [Requirements & Setup](#4-requirements--setup)
5. [Mongoose Schema Definitions for C0de Front](#5-mongoose-schema-definitions-for-c0de-front)
6. [High Level Architecture](#6-high-level-architecture)
7. [Backend Flow](#7-backend-flow)
8. [FrontEnd](#8-frontend)

---


# 1. Introduction

**C0de Front** is a LeetCode-style coding platform developed to help users practice DSA problems with a robust full-stack experience. It features:

- A backend for problem management, user authentication, and live code execution.
- A frontend with:
  - A built-in code editor (Monaco)
  - Chat-based AI assistant (Gemini API)
  - Upcoming Cloudinary video upload integration

---

# 2. Problem Statement

The goal was to build a coding platform similar to LeetCode where users can:

- Solve coding problems online
- Track their progress and submissions
- Get real-time help via an AI assistant
- Upload video solutions and explanations

**Key challenges included:**

- Building a secure and scalable backend
- Efficient schema design for DSA problems and user data
- Code execution sandbox integration
- Responsive and interactive frontend
- AI and multimedia integration

---

---
# 3. Features

### 🔐 Authentication & Authorization
- Secure **user authentication** using **JWT** tokens and **bcrypt** password hashing.
- **Role-based access control** (`admin`, `user`) implemented via custom middleware.
- **Token blacklisting** using **Redis** ensures secure logout and session handling.

### 👤 User Functionality
- **Signup/Login** system with JWT authentication.
- Users can:
  - **View all coding problems** or filter only their **solved problems**.
  - **Run** or **Submit** code for any problem in **Java, C++, or JavaScript**.
  - **Delete their profile**, which also removes all related submissions.

### 🧑‍💼 Admin Functionality
- Full problem management capabilities:
  - **Create**, **update**, and **delete** coding problems.
  - Add **problem metadata**: title, description, difficulty level (`easy`, `medium`, `hard`), and topic tags (`array`, `graph`, etc.).
  - Add both **visible and hidden test cases**.
  - Provide **boilerplate code** and **reference solutions** for each supported language.

### 🧪 Code Execution & Evaluation
- **Judge0 API integration** to compile and run user code.
- **Run mode**: Executes code without saving submission.
- **Submit mode**: Executes code against all test cases and saves the result in the database.
- All user code input is **sanitized** to avoid execution errors.

### 🧠 AI-Powered Chatbot (via Gemini API)
- Integrated an **AI assistant** to help users with **DSA-related doubts**.
- Enhancements for contextual and safe interactions:
  - **Max output token limit** of 500 to manage resource usage.
  - **System Instructions** restrict responses strictly to coding/problem-solving topics.
  - Automatically passes the **current problem’s title, description, test cases, and boilerplate** as context.
  - Enables users to ask vague queries like _“help me with this problem”_ and receive accurate, relevant help.

### 🗃️ Database Design
- **Three core Mongoose schemas**:
  - `User`: Includes role, problem history, and secure fields.
  - `Problem`: Fully structured problem model with test cases, tags, and more.
  - `Submission`: Stores every code submission, its result, runtime, and metadata.
- **Compound indexing** on `(userId, problemId)` in the `Submission` schema for efficient querying.

### 🔮 Future Enhancements
- **Video solution support** via **video streaming integration** is planned in the next phase.
  
---


# 4. Requirements & Setup

Make sure you have **Node.js**, **npm**, and **MongoDB** installed.

Then, install the following packages:

## 📦 Backend Dependencies

Install the following packages to set up the backend server.

### Tech Stack: Node.js, Express.js, MongoDB, Mongoose, Cookie-Parser, Dotenv, JSON Web Token, Validator, Redis, Axios, Bcrypt, CORS
```
npm install express mongoose cookie-parser dotenv jsonwebtoken validator redis axios bcrypt cors
```

## 💻 Frontend Dependencies

Install these packages to set up the frontend using React and Tailwind CSS.

### Tech Stack: React, Redux, JSX, Tailwind CSS, DaisyUI, Vite, react-hook-form, Zod, @hookform/resolvers, @monaco-editor/react, @google/genai, react-markdown, remark-gfm, rehype-highlight
```
npm install react redux tailwindcss daisyui vite react-hook-form zod @hookform/resolvers @monaco-editor/react @google/genai react-markdown remark-gfm rehype-highlight
```

---


# 5. Mongoose Schema Definitions for C0de Front

This  outlines the MongoDB (Mongoose) schema definitions used in the **C0de Front** application.


## 🧑 User Schema

### Fields
```js
firstName:        { type: String, required: true, minLength: 2, maxLength: 20 },
lastName:         { type: String, minLength: 3, maxLength: 20 },
emailID:          { type: String, required: true, unique: true, trim: true, lowercase: true, immutable: true },
age:              { type: Number, min: 5, max: 80 },
role:             { type: String, enum: ['user', 'admin'], default: 'user' },
problemSolved:    [{ type: Schema.Types.ObjectId, ref: 'problem' }],
password:         { type: String, required: true }
```

### Options
```js
{ timestamps: true }
```

### Post Hook
```js
userSchema.post('findOneAndDelete', async function (userInfo) {
  if (userInfo) {
    await mongoose.model('submission').deleteMany({ userId: userInfo._id });
  }
});
```


## 📤 Submission Schema

### Fields
```js
userId:           { type: Schema.Types.ObjectId, ref: 'user', required: true, index: true },
problemId:        { type: Schema.Types.ObjectId, ref: 'problem', required: true, index: true },
code:             { type: String, required: true },
language:         { type: String, required: true, enum: ['javascript', 'c++', 'java'] },
status:           { type: String, enum: ['pending', 'accepted', 'wrong answer', 'time limit exceeded', 'compilation error', 'runtime error', 'error'], default: 'pending' },
runtime:          { type: Number, default: 0 },   // in milliseconds
memory:           { type: Number, default: 0 },   // in KB
errorMessage:     { type: String, default: '' },
testCasesPassed:  { type: Number, default: 0 },
testCasesTotal:   { type: Number, default: 0 }
```

### Options
```js
{ timestamps: true }
```

### Compound Indexing
```js
submissionSchema.index({ userId: 1, problemId: 1 });
```


## 📚 Problem Schema

### Fields
```js
title:            { type: String, required: true },
description:      { type: String, required: true },
difficulty:       { type: String, enum: ['easy', 'medium', 'hard'], required: true },
tags:             { type: String, enum: ['array', 'linkedList', 'graph', 'dp'], required: true },
visibleTestCases: [{
  input:          { type: String, required: true },
  output:         { type: String, required: true },
  explanation:    { type: String, required: true }
}],
hiddenTestCases: [{
  input:          { type: String, required: true },
  output:         { type: String, required: true }
}],
startCode: [{
  language:       { type: String, required: true },
  initialCode:    { type: String, required: true }
}],
referenceSolution: [{
  language:       { type: String, required: true },
  completeCode:   { type: String, required: true }
}],
problemCreator:    { type: Schema.Types.ObjectId, ref: 'user', required: true }
```

### Options
```js
{ timestamps: true }
```

## 🎥 VideoSolution Schema

### Fields
```js
problemId:         { type: Schema.Types.ObjectId, ref: 'problem', required: true },
userId:            { type: Schema.Types.ObjectId, ref: 'user', required: true },
cloudinaryPublicId:{ type: String, required: true, unique: true },
secureUrl:         { type: String, required: true },
thumbnailUrl:      { type: String },
duration:          { type: Number, required: true }
```

### Options
```js
{ timestamps: true }
```

---

# 6. High Level Architecture

##  Overview

- **Full-Stack Platform**:  
  C0de Front is designed as a full-stack coding platform focused on DSA (Data Structures and Algorithms) practice.

- **Frontend (React)**:
  - Built using **React** with **Monaco Editor** for code input.
  - **Redux** is used for global state management.
  - UI styled using **Tailwind CSS** and **DaisyUI**.
  - Communicates with the backend through **HTTP/REST APIs**.

- **Backend (Node.js/Express)**:
  - Handles all business logic, route handling, and integration with third-party services.
  - Protected with **JWT-based authentication middleware** and **rate limiters**.

- **Database (MongoDB)**:
  - Stores data using four core schemas:
    - `User`
    - `Problem`
    - `Submission`
    - `VideoSolution` *(upcoming)*
  - Efficient querying via **indexing** (e.g., compound index on `userId` + `problemId`).

- **Third-Party Integrations**:
  - 🧠 **Gemini API**: Provides AI chatbot support for DSA-related queries.
  - 🧪 **Judge0 API**: Compiles and executes submitted code against visible and hidden test cases.
  - 📹 **Cloudinary** *(in progress)*: Will store video solutions submitted by users.

- **Security & Middleware**:
  - **JWT authentication** ensures protected routes.
  - **Rate limiting** helps prevent abuse (especially for code submission).

- **Deployment**:
  - Currently running **locally**:
    - Backend on a local Node.js server.
    - Frontend hosting planned (e.g., **Netlify**, **Vercel**, etc.).
  - Assumes a **stable internet connection** for external API calls.
  - Not yet optimized for high traffic (acknowledged area of growth).

- **Vision**:
  - Provide users with the ability to:
    - Practice coding problems.
    - Track their progress.
    - Receive AI-based help.
    - Access future features like **video streaming** for solutions.

  <img src="images/HLD.png" alt="HLD" width="100%" />

---

# 7. Backend Flow
### Working of Authorization
```mermaid
sequenceDiagram
  actor User
  participant Client
  participant userMiddleware
  participant isAdminMiddleware
  participant Handler
  participant MongoDB
  participant Redis

  User ->> Client: Send request (POST /user/register)
  Client ->> Handler: Direct to register/login/logout
  Handler ->> MongoDB: Save/verify user data
  Handler ->> Client: Return success/failure

  User ->> Client: POST /user/logout
  Client ->> userMiddleware: Verify JWT
  userMiddleware ->> Redis: Blacklist token
  userMiddleware ->> Client: Pass to Handler
  Handler ->> Client: Confirm logout

  User ->> Client: POST /user/admin/register
  Client ->> isAdminMiddleware: Check admin role
  isAdminMiddleware ->> MongoDB: Verify admin
  isAdminMiddleware ->> Client: Pass/Reject
  Handler ->> MongoDB: Register admin
  Handler ->> Client: Return response

  User ->> Client: DELETE /user/deleteprofile
  Client ->> userMiddleware: Verify JWT
  userMiddleware ->> MongoDB: Delete user/submissions
  userMiddleware ->> Client: Return success
```

### Working of Problem Creation/Updation/Deletion/Viewing
```mermaid
sequenceDiagram
    actor User
    participant Client
    participant isAdminMiddleware as AdminMW
    participant userMiddleware as UserMW
    participant Handler
    participant MongoDB as DB

    User ->> Client: POST /problem/create
    Client ->> AdminMW: Verify admin role
    AdminMW ->> DB: Create new problem (title, description, etc.)
    AdminMW ->> Client: Return success/failure

    User ->> Client: PUT /problem/update/:id
    Client ->> AdminMW: Verify admin role
    AdminMW ->> DB: Update problem by ID
    AdminMW ->> Client: Return success/failure

    User ->> Client: DELETE /problem/delete/:id
    Client ->> AdminMW: Verify admin role
    AdminMW ->> DB: Delete problem by ID
    AdminMW ->> Client: Return success/failure

    User ->> Client: GET /problem/getAllProblem
    Client ->> UserMW: Verify user
    UserMW ->> DB: Fetch all problems
    UserMW ->> Client: Return problem list

    User ->> Client: GET /problem/problemById/:id
    Client ->> UserMW: Verify user
    UserMW ->> DB: Fetch problem by ID
    UserMW ->> Client: Return problem data

    User ->> Client: GET /problem/problemSolvedByUser
    Client ->> UserMW: Verify user
    UserMW ->> DB: Fetch solved problems by user
    UserMW ->> Client: Return solved list

    User ->> Client: GET /problem/submittedProblem/:pid
    Client ->> UserMW: Verify user
    UserMW ->> DB: Fetch submissions for problem by user
    UserMW ->> Client: Return submission data
```

### Working of Submission
```mermaid
sequenceDiagram
    actor User
    participant Client
    participant userMiddleware as UserMW
    participant submitCodeRateLimiter as RateLimit
    participant Handler
    participant Judge0_API as JudgeAPI
    participant MongoDB as DB

    %% Submit Code Flow
    User ->> Client: POST /submission/submit/:id
    Client ->> UserMW: Verify JWT token
    UserMW ->> RateLimit: Check submission rate
    RateLimit ->> Handler: Pass if within limit
    Handler ->> JudgeAPI: Submit code for evaluation
    JudgeAPI ->> Handler: Return status, runtime, memory
    Handler ->> DB: Save submission details
    Handler ->> Client: Return result (status, error)

    %% Run Code Flow
    User ->> Client: POST /submission/run/:id
    Client ->> UserMW: Verify JWT token
    UserMW ->> RateLimit: Check submission rate
    RateLimit ->> Handler: Pass if within limit
    Handler ->> JudgeAPI: Run code
    JudgeAPI ->> Handler: Return output or error
    Handler ->> Client: Return result (output, error)
```
### Working of AI Chatbot
```mermaid
sequenceDiagram
    actor User
    participant Client
    participant userMiddleware as UserMW
    participant Handler
    participant Gemini_API as GeminiAPI
    participant MongoDB as DB

    User ->> Client: POST /ai/chat (with chat data)
    Client ->> UserMW: Verify JWT token
    UserMW ->> DB: Check user existence
    UserMW ->> Client: Pass if valid, reject if invalid
    Client ->> Handler: Forward request
    Handler ->> GeminiAPI: Send chat data (role/parts)
    GeminiAPI ->> Handler: Return DSA response
    Handler ->> Client: Return response (DSA solution)
    Client ->> User: Display response
```

---

# 8. FrontEnd

As someone opens the site he/she is redirected to signup page, if not authorized. The user can choose to either signup or login.

<p align="center">
  <img src="images/signup.png" alt="Signup Page" width="45%" />
  <img src="images/login.png" alt="Login Page" width="45%" />
</p>

---

After signup/login user lands on homepage. On homepage user can see the problems, his/her progress, filter such as Diffculty, Topic, All/Solved problems to filter questions.
Left image shows homepage in dark mode, right image shows homepage with filters applied (tag: DP, difficulty: hard)

<p align="center">
  <img src="images/homepage_dark.png" alt="Signup Page" width="45%" />
  <img src="images/homepage_light_filters.png" alt="Login Page" width="45%" />
</p>

---

If the user is an admin he/she can Create/Update/Delete problems

<img src="images/admin_panel.png" alt="Admin Page" width="65%" />

Admin can create a problem by filling details of 'title', 'description', 'difficulty', 'problem tag', 'visible test-cases', 'hidden test-cases', 'bolierplate code and reference solution for C++, Java & Javascript'.

<img src="images/admin_create.png" alt="Admin Create Page" width="65%" />

Admin can update a particular problem.

<img src="images/admin_update_1.png" alt="Admin Update Page" width="65%" />

Admin can update a problem or it's details. The details will be fetched from database via backend which admin can update.

<p align="center">
<img src="images/admin_update_2.1.png" alt="Admin Update Form1 Page" width="45%" />
<img src="images/admin_update_2.2.png" alt="Admin Update Form2 Page" width="45%" />
</p>

---

Admin can delete a particular problem.

<img src="images/admin_delete.png" alt="Admin Delete Page" width="65%" />

---

Apart from this any user (normal or admin) can attempt any question. After clicking on any question from homepage, user will be directed to problem page.
The page contains Description, Editorial, Submissions, Solutions, AI Chatbot on left panel and Code editor on Right Panel. 

Description section:

<p align="center">
<img src="images/user_problem_description_light.png" alt="Problem Page" width="45%" />
<img src="images/user_problem_description_dark.png" alt="Problem Page" width="45%" />
</p>

---

Solutions section:

<img src="images/user_problem_solutions.png" alt="Problem Page" width="65%" />

---

Submissions section:

<p align="center">
<img src="images/user_problem_submissions.png" alt="Problem Page" width="45%" />
<img src="images/user_problem_submissions2.png" alt="Problem Page" width="45%" />
</p>

---

AI Chatbot section (the bold text and code are formatted using ReactMarkDown):

<p align="center">
<img src="images/user_problem_ai_dark.png" alt="Problem Page" width="45%" />
<img src="images/user_problem_ai_light.png" alt="Problem Page" width="45%" />
</p>

---

After submitting the code:

<img src="images/user_problem_submit.png" alt="Problem Page" width="65%" />

---

THIS WRAPS UP IT ALL. I am curently working on adding video feature via cloudinary using streaming method
