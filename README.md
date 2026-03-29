# user_management_system_
The rapid growth of web-based applications has increased the need for secure and scalable user management systems. Modern businesses require systems that can efficiently handle user authentication, authorization, and role-based access.
server.js
JavaScript
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const authRoutes = require('./routes/auth');

require('dotenv').config();

const app = express();
app.use(express.json());
app.use(cors());

mongoose.connect(process.env.MONGO_URI)
.then(() => console.log("MongoDB Connected"))
.catch(err => console.log(err));

app.use('/api/auth', authRoutes);

app.listen(5000, () => console.log("Server running on port 5000"));
📄 models/User.js
JavaScript
const mongoose = require('mongoose');

const UserSchema = new mongoose.Schema({
  name: String,
  email: String,
  password: String,
  role: {
    type: String,
    enum: ['user', 'admin'],
    default: 'user'
  }
});

module.exports = mongoose.model('User', UserSchema);
📄 routes/auth.js
JavaScript
const router = require('express').Router();
const User = require('../models/User');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');

// REGISTER
router.post('/register', async (req, res) => {
  const { name, email, password } = req.body;

  const hashedPassword = await bcrypt.hash(password, 10);

  const newUser = new User({
    name,
    email,
    password: hashedPassword
  });

  await newUser.save();
  res.json("User Registered");
});

// LOGIN
router.post('/login', async (req, res) => {
  const user = await User.findOne({ email: req.body.email });
  if (!user) return res.status(400).json("User not found");

  const valid = await bcrypt.compare(req.body.password, user.password);
  if (!valid) return res.status(400).json("Invalid password");

  const token = jwt.sign(
    { id: user._id, role: user.role },
    "secretkey",
    { expiresIn: "1h" }
  );

  res.json({ token, role: user.role });
});

module.exports = router;
🎨 2. Frontend (React)
📁 Create React App
Bash
npx create-react-app client
cd client
npm install axios react-router-dom
📄 App.js
JavaScript
import React from 'react';
import Login from './Login';
import Register from './Register';

function App() {
  return (
    <div>
      <h1>User Management System</h1>
      <Login />
      <Register />
    </div>
  );
}

export default App;
📄 Login.js
JavaScript
import React, { useState } from 'react';
import axios from 'axios';

function Login() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleLogin = async () => {
    const res = await axios.post('http://localhost:5000/api/auth/login', {
      email,
      password
    });

    localStorage.setItem('token', res.data.token);
    alert("Login Successful");
  };

  return (
    <div>
      <h2>Login</h2>
      <input placeholder="Email" onChange={e => setEmail(e.target.value)} />
      <input type="password" placeholder="Password" onChange={e => setPassword(e.target.value)} />
      <button onClick={handleLogin}>Login</button>
    </div>
  );
}

export default Login;
📄 Register.js
JavaScript
import React, { useState } from 'react';
import axios from 'axios';

function Register() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');

  const handleRegister = async () => {
    await axios.post('http://localhost:5000/api/auth/register', {
      name,
      email,
      password
    });

    alert("Registered Successfully");
  };

  return (
    <div>
      <h2>Register</h2>
      <input placeholder="Name" onChange={e => setName(e.target.value)} />
      <input placeholder="Email" onChange={e => setEmail(e.target.value)} />
      <input type="password" placeholder="Password" onChange={e => setPassword(e.target.value)} />
      <button onClick={handleRegister}>Register</button>
    </div>
  );
}

export default Register;
🚀 3. How to Run
Backend:
Bash
node server.js
Frontend:
Bash
npm start
