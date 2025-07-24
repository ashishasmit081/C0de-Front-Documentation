# 🚀 C0de Front

## 🧠 Introduction

**C0de Front** is a LeetCode-style coding platform developed to help users practice DSA problems with a robust full-stack experience. It features:

- A backend for problem management, user authentication, and live code execution.
- A frontend with:
  - A built-in code editor (Monaco)
  - Chat-based AI assistant (Gemini API)
  - Upcoming Cloudinary video upload integration

---

## 🧩 Problem Statement

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

## ⚙️ Requirements & Setup

Make sure you have **Node.js**, **npm**, and **MongoDB** installed.

Then, install the following packages:

## 📦 Backend Dependencies

Install the following packages to set up the backend server.

### Tech Stack: Node.js, Express.js, MongoDB, Mongoose, Cookie-Parser, Dotenv, JSON Web Token, Validator, Redis, Axios, Bcrypt, CORS
```
npm install express mongoose cookie-parser dotenv jsonwebtoken validator redis axios bcrypt cors
```

---

## 💻 Frontend Dependencies

Install these packages to set up the frontend using React and Tailwind CSS.

### Tech Stack: React, Redux, JSX, Tailwind CSS, DaisyUI, Vite, react-hook-form, Zod, @hookform/resolvers, @monaco-editor/react, @google/genai, react-markdown, remark-gfm, rehype-highlight
```
npm install react redux tailwindcss daisyui vite react-hook-form zod @hookform/resolvers @monaco-editor/react @google/genai react-markdown remark-gfm rehype-highlight
```

---

# 📘 Mongoose Schema Definitions for C0de Front

This  outlines the MongoDB (Mongoose) schema definitions used in the **C0de Front** application.

---

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

---

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

### Indexing
```js
submissionSchema.index({ userId: 1, problemId: 1 });
```

---

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

---

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
