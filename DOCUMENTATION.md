# 📚 Notes Keeper - Complete Project Documentation

## Table of Contents
1. [Project Overview](#project-overview)
2. [System Architecture](#system-architecture)
3. [Technology Stack](#technology-stack)
4. [Database Design](#database-design)
5. [Backend Implementation](#backend-implementation)
6. [Frontend Implementation](#frontend-implementation)
7. [Authentication & Security](#authentication--security)
8. [API Reference](#api-reference)
9. [Deployment Guide](#deployment-guide)
10. [Testing](#testing)
11. [Troubleshooting](#troubleshooting)

---

## Project Overview

### Introduction
Notes Keeper is a full-stack web application designed to provide users with a seamless note-taking experience. Built using the MERN (MongoDB, Express.js, React, Node.js) stack, it offers a modern, responsive interface inspired by Google Keep.

### Project Goals
- Provide secure, user-specific note management
- Enable real-time search and filtering capabilities
- Offer intuitive UI/UX with theme customization
- Implement robust authentication and authorization
- Ensure scalability and maintainability

### Key Features
- User authentication with JWT
- CRUD operations for notes
- Real-time search functionality
- Pin important notes
- Archive and trash management
- Dark/Light theme toggle
- Responsive design for all devices
- Markdown support

---

## System Architecture

### High-Level Architecture

```
┌─────────────────┐         ┌─────────────────┐         ┌─────────────────┐
│                 │         │                 │         │                 │
│  React Frontend │ ◄─────► │  Express API    │ ◄─────► │  MongoDB        │
│  (Port 3000)    │  HTTP   │  (Port 5000)    │  CRUD   │  Database       │
│                 │         │                 │         │                 │
└─────────────────┘         └─────────────────┘         └─────────────────┘
```

### Application Flow

1. **User Registration/Login**
   - User submits credentials
   - Backend validates and hashes password
   - JWT token generated and returned
   - Token stored in localStorage

2. **Note Operations**
   - User performs CRUD operations
   - Frontend sends authenticated requests
   - Backend validates JWT token
   - Database operations executed
   - Response sent back to frontend

3. **Real-time Search**
   - User types in search bar
   - Frontend sends search query
   - Backend performs MongoDB text search
   - Filtered results returned

---

## Technology Stack

### Frontend Technologies

#### React (18.2.0)
- **Purpose**: UI library for building component-based interface
- **Why chosen**: Virtual DOM for performance, component reusability, large ecosystem

#### Axios (1.5.0)
- **Purpose**: HTTP client for API requests
- **Why chosen**: Promise-based, interceptors support, automatic JSON transformation

#### Marked (9.1.6)
- **Purpose**: Markdown parser and compiler
- **Why chosen**: Fast, lightweight, supports GitHub Flavored Markdown

#### CSS3
- **Purpose**: Styling and animations
- **Features used**: Flexbox, Grid, CSS Variables, Transitions, Media Queries

### Backend Technologies

#### Node.js (≥18.0.0)
- **Purpose**: JavaScript runtime environment
- **Why chosen**: Non-blocking I/O, JavaScript everywhere, npm ecosystem

#### Express.js (4.18.2)
- **Purpose**: Web application framework
- **Why chosen**: Minimalist, flexible, robust routing, middleware support

#### MongoDB
- **Purpose**: NoSQL database
- **Why chosen**: Flexible schema, scalability, JSON-like documents

#### Mongoose (7.5.0)
- **Purpose**: MongoDB ODM (Object Data Modeling)
- **Why chosen**: Schema validation, middleware, query building

### Security & Authentication

#### JWT (jsonwebtoken 9.0.2)
- **Purpose**: Token-based authentication
- **Why chosen**: Stateless, scalable, secure

#### bcryptjs (2.4.3)
- **Purpose**: Password hashing
- **Why chosen**: Secure, salt rounds configurable, widely trusted

#### CORS
- **Purpose**: Cross-Origin Resource Sharing
- **Why chosen**: Enable frontend-backend communication

### Development Tools

#### Nodemon (3.0.1)
- **Purpose**: Auto-restart server on file changes
- **Why chosen**: Improves development workflow

#### dotenv (16.3.1)
- **Purpose**: Environment variable management
- **Why chosen**: Secure configuration, easy deployment

---

## Database Design

### Database Schema

#### Users Collection
```javascript
{
  _id: ObjectId,
  name: String (required),
  email: String (required, unique, lowercase),
  password: String (required, hashed),
  createdAt: Date (auto-generated),
  updatedAt: Date (auto-generated)
}
```

**Indexes:**
- `email`: Unique index for fast lookup and preventing duplicates

#### Notes Collection
```javascript
{
  _id: ObjectId,
  title: String (required),
  content: String (required),
  isPinned: Boolean (default: false),
  isArchived: Boolean (default: false),
  isDeleted: Boolean (default: false),
  color: String (default: 'default'),
  userId: ObjectId (required, ref: 'User'),
  createdAt: Date (auto-generated),
  updatedAt: Date (auto-generated)
}
```

**Indexes:**
- `userId`: Index for filtering user-specific notes
- `title, content`: Text index for search functionality

### Relationships
- **One-to-Many**: One User can have many Notes
- **Foreign Key**: `userId` in Notes references `_id` in Users

---

## Backend Implementation

### Project Structure
```
backend/
├── middleware/
│   └── auth.js              # JWT authentication middleware
├── models/
│   ├── User.js              # User schema and model
│   └── Note.js              # Note schema and model
├── routes/
│   ├── auth.js              # Authentication routes
│   └── notes.js             # Notes CRUD routes
├── .env                     # Environment variables
├── server.js                # Application entry point
└── package.json             # Dependencies and scripts
```

### Server Configuration (server.js)

```javascript
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
require('dotenv').config();

const app = express();

// Middleware
app.use(cors());
app.use(express.json());

// Database Connection
mongoose.connect(process.env.MONGODB_URI)
  .then(() => console.log('MongoDB connected'))
  .catch(err => console.error('MongoDB connection error:', err));

// Routes
app.use('/api/auth', require('./routes/auth'));
app.use('/api/notes', require('./routes/notes'));

// Start Server
const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

### Authentication Middleware (middleware/auth.js)

**Purpose**: Verify JWT tokens and protect routes

```javascript
const jwt = require('jsonwebtoken');

module.exports = function(req, res, next) {
  // Get token from header
  const token = req.header('Authorization')?.replace('Bearer ', '');
  
  if (!token) {
    return res.status(401).json({ message: 'No token, authorization denied' });
  }
  
  try {
    // Verify token
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.userId = decoded.userId;
    next();
  } catch (err) {
    res.status(401).json({ message: 'Token is not valid' });
  }
};
```

### User Model (models/User.js)

```javascript
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  name: { type: String, required: true },
  email: { type: String, required: true, unique: true, lowercase: true },
  password: { type: String, required: true }
}, { timestamps: true });

module.exports = mongoose.model('User', userSchema);
```

### Note Model (models/Note.js)

```javascript
const mongoose = require('mongoose');

const noteSchema = new mongoose.Schema({
  title: { type: String, required: true },
  content: { type: String, required: true },
  isPinned: { type: Boolean, default: false },
  isArchived: { type: Boolean, default: false },
  isDeleted: { type: Boolean, default: false },
  color: { type: String, default: 'default' },
  userId: { type: mongoose.Schema.Types.ObjectId, ref: 'User', required: true }
}, { timestamps: true });

// Text index for search
noteSchema.index({ title: 'text', content: 'text' });

module.exports = mongoose.model('Note', noteSchema);
```

### Authentication Routes (routes/auth.js)

**Endpoints:**
- `POST /api/auth/signup` - Register new user
- `POST /api/auth/login` - Login user

**Key Features:**
- Password hashing with bcrypt
- JWT token generation
- Email validation
- Duplicate email prevention

### Notes Routes (routes/notes.js)

**Endpoints:**
- `GET /api/notes` - Get all user notes (with search)
- `POST /api/notes` - Create new note
- `PUT /api/notes/:id` - Update note
- `DELETE /api/notes/:id` - Delete note

**Key Features:**
- User-specific note filtering
- Search functionality
- Authorization checks
- Error handling

---

## Frontend Implementation

### Project Structure
```
frontend/
├── public/
│   └── index.html           # HTML template
├── src/
│   ├── App.js               # Main application component
│   ├── Auth.js              # Authentication component
│   ├── NoteCard.js          # Individual note display
│   ├── NoteForm.js          # Note creation/editing form
│   ├── ConfirmDialog.js     # Confirmation modal
│   ├── Footer.js            # Footer component
│   ├── api.js               # API service layer
│   ├── index.js             # React entry point
│   └── index.css            # Global styles
├── .env.example             # Environment variables template
└── package.json             # Dependencies and scripts
```

### Component Architecture

#### App.js (Main Component)
**Responsibilities:**
- State management (notes, user, theme)
- Authentication handling
- Note CRUD operations
- Search functionality
- View switching (all/archive/trash)

**Key State:**
```javascript
const [notes, setNotes] = useState([]);
const [user, setUser] = useState(null);
const [token, setToken] = useState(localStorage.getItem('token'));
const [searchQuery, setSearchQuery] = useState('');
const [theme, setTheme] = useState('dark');
const [view, setView] = useState('all');
```

#### Auth.js (Authentication)
**Features:**
- Login/Signup form toggle
- Form validation
- Error handling
- Token storage

#### NoteCard.js (Note Display)
**Features:**
- Note preview
- Pin/Unpin toggle
- Archive/Unarchive
- Delete/Restore
- Color selection
- Edit mode

#### NoteForm.js (Note Editor)
**Features:**
- Title and content input
- Markdown preview
- Save/Cancel actions
- Auto-focus

### API Service Layer (api.js)

```javascript
import axios from 'axios';

const API_URL = process.env.REACT_APP_API_URL || 'http://localhost:5000/api';

// Create axios instance with default config
const api = axios.create({
  baseURL: API_URL,
  headers: { 'Content-Type': 'application/json' }
});

// Add token to requests
api.interceptors.request.use(config => {
  const token = localStorage.getItem('token');
  if (token) {
    config.headers.Authorization = `Bearer ${token}`;
  }
  return config;
});

export default api;
```

### Styling Approach

**CSS Architecture:**
- CSS Variables for theming
- BEM-like naming convention
- Mobile-first responsive design
- Flexbox and Grid layouts

**Theme Implementation:**
```css
:root {
  --bg-primary: #1a1a2e;
  --bg-secondary: #16213e;
  --text-primary: #ffffff;
  --accent: #667eea;
}

[data-theme="light"] {
  --bg-primary: #f5f5f5;
  --bg-secondary: #ffffff;
  --text-primary: #333333;
}
```

---

## Authentication & Security

### Password Security

**Hashing Process:**
1. User submits password
2. bcrypt generates salt (10 rounds)
3. Password hashed with salt
4. Hash stored in database

```javascript
const bcrypt = require('bcryptjs');
const salt = await bcrypt.genSalt(10);
const hashedPassword = await bcrypt.hash(password, salt);
```

### JWT Implementation

**Token Structure:**
```javascript
{
  userId: user._id,
  iat: issuedAt,
  exp: expiresAt
}
```

**Token Generation:**
```javascript
const token = jwt.sign(
  { userId: user._id },
  process.env.JWT_SECRET,
  { expiresIn: '7d' }
);
```

**Token Verification:**
```javascript
const decoded = jwt.verify(token, process.env.JWT_SECRET);
```

### Security Best Practices

1. **Environment Variables**: Sensitive data in .env
2. **HTTPS**: Use SSL/TLS in production
3. **CORS Configuration**: Restrict allowed origins
4. **Input Validation**: Sanitize user inputs
5. **Rate Limiting**: Prevent brute force attacks
6. **Password Requirements**: Minimum length, complexity

---

## API Reference

### Base URL
```
Development: http://localhost:5000/api
Production: https://your-api-domain.com/api
```

### Authentication Endpoints

#### Register User
```http
POST /auth/signup
Content-Type: application/json

Request Body:
{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "securePassword123"
}

Success Response (201):
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": "507f1f77bcf86cd799439011",
    "name": "John Doe",
    "email": "john@example.com"
  }
}

Error Response (400):
{
  "message": "User already exists"
}
```

#### Login User
```http
POST /auth/login
Content-Type: application/json

Request Body:
{
  "email": "john@example.com",
  "password": "securePassword123"
}

Success Response (200):
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": "507f1f77bcf86cd799439011",
    "name": "John Doe",
    "email": "john@example.com"
  }
}

Error Response (401):
{
  "message": "Invalid credentials"
}
```

### Notes Endpoints (Protected)

**Authentication Required**: All endpoints require JWT token in Authorization header
```
Authorization: Bearer <your_jwt_token>
```

#### Get All Notes
```http
GET /notes?search=keyword&view=all
Query Parameters:
- search (optional): Search term for title/content
- view (optional): all | archive | trash

Success Response (200):
{
  "notes": [
    {
      "_id": "507f1f77bcf86cd799439011",
      "title": "My Note",
      "content": "Note content here",
      "isPinned": false,
      "isArchived": false,
      "isDeleted": false,
      "color": "default",
      "userId": "507f1f77bcf86cd799439012",
      "createdAt": "2024-01-15T10:30:00.000Z",
      "updatedAt": "2024-01-15T10:30:00.000Z"
    }
  ]
}
```

#### Create Note
```http
POST /notes
Content-Type: application/json

Request Body:
{
  "title": "My Note",
  "content": "Note content here",
  "isPinned": false,
  "color": "blue"
}

Success Response (201):
{
  "note": {
    "_id": "507f1f77bcf86cd799439011",
    "title": "My Note",
    "content": "Note content here",
    "isPinned": false,
    "color": "blue",
    "userId": "507f1f77bcf86cd799439012",
    "createdAt": "2024-01-15T10:30:00.000Z"
  }
}
```

#### Update Note
```http
PUT /notes/:id
Content-Type: application/json

Request Body:
{
  "title": "Updated Title",
  "content": "Updated content",
  "isPinned": true,
  "isArchived": false
}

Success Response (200):
{
  "note": { /* updated note object */ }
}

Error Response (404):
{
  "message": "Note not found"
}
```

#### Delete Note
```http
DELETE /notes/:id

Success Response (200):
{
  "message": "Note deleted successfully"
}

Error Response (404):
{
  "message": "Note not found"
}
```

---

## Deployment Guide

### Prerequisites
- GitHub account
- MongoDB Atlas account
- Vercel account (frontend)
- Render/Railway account (backend)

### Step 1: Database Setup (MongoDB Atlas)

1. **Create Cluster**
   - Go to [MongoDB Atlas](https://www.mongodb.com/cloud/atlas)
   - Create free cluster
   - Choose cloud provider and region

2. **Configure Access**
   - Database Access: Create database user
   - Network Access: Add IP address (0.0.0.0/0 for all)

3. **Get Connection String**
   - Click "Connect" → "Connect your application"
   - Copy connection string
   - Replace `<password>` with your database user password

### Step 2: Backend Deployment (Render)

1. **Push Code to GitHub**
   ```bash
   git add .
   git commit -m "Prepare for deployment"
   git push origin main
   ```

2. **Create Web Service on Render**
   - Go to [Render Dashboard](https://dashboard.render.com/)
   - Click "New +" → "Web Service"
   - Connect GitHub repository
   - Select backend directory

3. **Configure Service**
   - Name: `notekeeper-api`
   - Environment: `Node`
   - Build Command: `npm install`
   - Start Command: `npm start`
   - Instance Type: Free

4. **Set Environment Variables**
   ```
   PORT=5000
   MONGODB_URI=mongodb+srv://username:password@cluster.mongodb.net/noteskeeper
   JWT_SECRET=your_super_secret_jwt_key_production
   NODE_ENV=production
   ```

5. **Deploy**
   - Click "Create Web Service"
   - Wait for deployment to complete
   - Copy service URL (e.g., `https://notekeeper-api.onrender.com`)

### Step 3: Frontend Deployment (Vercel)

1. **Update API URL**
   - Create `.env.production` file
   ```
   REACT_APP_API_URL=https://notekeeper-api.onrender.com/api
   ```

2. **Deploy to Vercel**
   - Go to [Vercel Dashboard](https://vercel.com/dashboard)
   - Click "Add New" → "Project"
   - Import GitHub repository
   - Select frontend directory

3. **Configure Project**
   - Framework Preset: Create React App
   - Root Directory: `Notepad/frontend`
   - Build Command: `npm run build`
   - Output Directory: `build`

4. **Set Environment Variables**
   ```
   REACT_APP_API_URL=https://notekeeper-api.onrender.com/api
   ```

5. **Deploy**
   - Click "Deploy"
   - Wait for deployment
   - Access your live application

### Step 4: Post-Deployment

1. **Update CORS Settings** (Backend)
   ```javascript
   app.use(cors({
     origin: 'https://your-frontend-url.vercel.app',
     credentials: true
   }));
   ```

2. **Test Application**
   - Register new user
   - Create notes
   - Test all features

3. **Monitor Performance**
   - Check Render logs
   - Monitor Vercel analytics
   - Set up error tracking (optional: Sentry)

---

## Testing

### Manual Testing Checklist

#### Authentication
- [ ] User can register with valid credentials
- [ ] Duplicate email registration is prevented
- [ ] User can login with correct credentials
- [ ] Invalid credentials show error message
- [ ] Token persists after page refresh
- [ ] Logout clears token

#### Notes CRUD
- [ ] User can create new note
- [ ] User can view all notes
- [ ] User can edit existing note
- [ ] User can delete note
- [ ] Deleted note moves to trash
- [ ] User can restore from trash
- [ ] User can permanently delete

#### Features
- [ ] Search filters notes correctly
- [ ] Pin/Unpin works properly
- [ ] Archive/Unarchive functions
- [ ] Color selection applies
- [ ] Theme toggle switches correctly
- [ ] Responsive on mobile devices

### API Testing with Postman

1. **Import Collection**
   - Create Postman collection
   - Add all API endpoints

2. **Test Authentication**
   ```
   POST {{baseUrl}}/auth/signup
   POST {{baseUrl}}/auth/login
   ```

3. **Test Notes Endpoints**
   - Set Authorization header with token
   - Test all CRUD operations

---

## Troubleshooting

### Common Issues

#### 1. MongoDB Connection Error
**Problem**: `MongooseServerSelectionError: connect ECONNREFUSED`

**Solutions:**
- Check MongoDB is running
- Verify connection string
- Check network access in MongoDB Atlas
- Ensure IP whitelist includes your IP

#### 2. CORS Error
**Problem**: `Access to XMLHttpRequest has been blocked by CORS policy`

**Solutions:**
- Enable CORS in backend
- Check allowed origins
- Verify API URL in frontend

#### 3. JWT Token Invalid
**Problem**: `Token is not valid`

**Solutions:**
- Check JWT_SECRET matches
- Verify token format
- Check token expiration
- Clear localStorage and re-login

#### 4. Port Already in Use
**Problem**: `Error: listen EADDRINUSE: address already in use :::5000`

**Solutions:**
```bash
# Find process using port
netstat -ano | findstr :5000

# Kill process
taskkill /PID <process_id> /F
```

#### 5. Build Errors
**Problem**: `Module not found` or dependency errors

**Solutions:**
```bash
# Clear cache and reinstall
rm -rf node_modules package-lock.json
npm install

# Clear React cache
npm start -- --reset-cache
```

### Debug Mode

**Enable Backend Logging:**
```javascript
// Add to server.js
app.use((req, res, next) => {
  console.log(`${req.method} ${req.path}`);
  next();
});
```

**Enable Frontend Logging:**
```javascript
// Add to api.js
api.interceptors.response.use(
  response => {
    console.log('API Response:', response);
    return response;
  },
  error => {
    console.error('API Error:', error.response);
    return Promise.reject(error);
  }
);
```

---

## Performance Optimization

### Backend Optimization
- Database indexing on frequently queried fields
- Implement pagination for large datasets
- Use MongoDB aggregation for complex queries
- Enable compression middleware
- Implement caching (Redis)

### Frontend Optimization
- Code splitting with React.lazy()
- Memoization with useMemo and useCallback
- Debounce search input
- Lazy load images
- Minimize bundle size

---

## Maintenance & Updates

### Regular Tasks
- Update dependencies monthly
- Monitor security vulnerabilities
- Backup database regularly
- Review and optimize queries
- Update documentation

### Version Control
- Use semantic versioning (MAJOR.MINOR.PATCH)
- Maintain CHANGELOG.md
- Tag releases in Git
- Document breaking changes

---

## Conclusion

This documentation provides a comprehensive guide to understanding, developing, deploying, and maintaining the Notes Keeper application. For additional support or questions, please refer to the main README.md or contact the development team.

**Last Updated**: January 2025
**Version**: 1.0.0
**Maintained by**: Manoj Kumar Sahoo
