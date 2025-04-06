# shyl// 📁 backend/server.js
const express = require("express");
const mongoose = require("mongoose");
const cors = require("cors");
const jwt = require("jsonwebtoken");
const bcrypt = require("bcryptjs");
require("dotenv").config();

const app = express();
app.use(cors());
app.use(express.json());

const PORT = process.env.PORT || 5000;

// MongoDB Bağlantısı
mongoose.connect(process.env.MONGO_URI, {
  useNewUrlParser: true,
  useUnifiedTopology: true,
})
.then(() => console.log("MongoDB bağlandı"))
.catch((err) => console.error("MongoDB bağlantı hatası:", err));

// Kullanıcı Modeli
const UserSchema = new mongoose.Schema({
  name: String,
  email: { type: String, unique: true },
  password: String,
  role: { type: String, enum: ["ogrenci", "ogretmen", "veli"] },
});

const User = mongoose.model("User", UserSchema);

// JWT Doğrulama Middleware
const authenticate = (req, res, next) => {
  const token = req.headers.authorization;
  if (!token) return res.status(401).json({ message: "Token eksik" });

  jwt.verify(token, process.env.JWT_SECRET, (err, decoded) => {
    if (err) return res.status(403).json({ message: "Geçersiz token" });
    req.user = decoded;
    next();
  });
};

// Auth Routes
app.post("/api/register", async (req, res) => {
  const { name, email, password, role } = req.body;
  const hashedPassword = await bcrypt.hash(password, 10);

  try {
    const newUser = await User.create({ name, email, password: hashedPassword, role });
    res.json({ message: "Kayıt başarılı" });
  } catch (err) {
    res.status(400).json({ error: "Kayıt başarısız", detail: err });
  }
});

app.post("/api/login", async (req, res) => {
  const { email, password } = req.body;
  const user = await User.findOne({ email });
  if (!user) return res.status(404).json({ error: "Kullanıcı bulunamadı" });

  const isPasswordValid = await bcrypt.compare(password, user.password);
  if (!isPasswordValid) return res.status(401).json({ error: "Hatalı şifre" });

  const token = jwt.sign({ id: user._id, role: user.role }, process.env.JWT_SECRET);
  res.json({ token, user: { id: user._id, name: user.name, role: user.role } });
});

// Test endpoint (protected)
app.get("/api/profile", authenticate, (req, res) => {
  res.json({ message: "Profil verisi", user: req.user });
});

app.listen(PORT, () => {
  console.log(`Sunucu ${PORT} portunda çalışıyor`);
});


// 📁 frontend/src/App.jsx
import React from "react";
import { useState, useEffect } from "react";

function App() {
  const [message, setMessage] = useState("");

  useEffect(() => {
    fetch("http://localhost:5000/")
      .then((res) => res.text())
      .then(setMessage);
  }, []);

  return (
    <div className="flex items-center justify-center min-h-screen bg-blue-100">
      <h1 className="text-2xl font-bold text-blue-700">{message}</h1>
    </div>
  );
}

export default App;


// 📁 frontend/src/main.jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App.jsx';
import './index.css';

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);


// 📁 frontend/index.html (Vite template)
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>EduNet</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>


// 📄 .env (backend klasörüne)
MONGO_URI=mongodb+srv://kullanici:parola@cluster.mongodb.net/edunet
PORT=5000
JWT_SECRET=supersecretkey123
cd backend
npm install
npm run dev  # ya da node server.js
// 📁 backend/server.js
const express = require("express");
const mongoose = require("mongoose");
const cors = require("cors");
const jwt = require("jsonwebtoken");
const bcrypt = require("bcryptjs");
require("dotenv").config();

const app = express();
app.use(cors());
app.use(express.json());

const PORT = process.env.PORT || 5000;

// MongoDB Bağlantısı
mongoose.connect(process.env.MONGO_URI, {
  useNewUrlParser: true,
  useUnifiedTopology: true,
})
.then(() => console.log("MongoDB bağlandı"))
.catch((err) => console.error("MongoDB bağlantı hatası:", err));

// Kullanıcı Modeli
const UserSchema = new mongoose.Schema({
  name: String,
  email: { type: String, unique: true },
  password: String,
  role: { type: String, enum: ["ogrenci", "ogretmen", "veli"] },
});

const User = mongoose.model("User", UserSchema);

// JWT Doğrulama Middleware
const authenticate = (req, res, next) => {
  const token = req.headers.authorization;
  if (!token) return res.status(401).json({ message: "Token eksik" });

  jwt.verify(token, process.env.JWT_SECRET, (err, decoded) => {
    if (err) return res.status(403).json({ message: "Geçersiz token" });
    req.user = decoded;
    next();
  });
};

// Auth Routes
app.post("/api/register", async (req, res) => {
  const { name, email, password, role } = req.body;
  const hashedPassword = await bcrypt.hash(password, 10);

  try {
    const newUser = await User.create({ name, email, password: hashedPassword, role });
    res.json({ message: "Kayıt başarılı" });
  } catch (err) {
    res.status(400).json({ error: "Kayıt başarısız", detail: err });
  }
});

app.post("/api/login", async (req, res) => {
  const { email, password } = req.body;
  const user = await User.findOne({ email });
  if (!user) return res.status(404).json({ error: "Kullanıcı bulunamadı" });

  const isPasswordValid = await bcrypt.compare(password, user.password);
  if (!isPasswordValid) return res.status(401).json({ error: "Hatalı şifre" });

  const token = jwt.sign({ id: user._id, role: user.role }, process.env.JWT_SECRET);
  res.json({ token, user: { id: user._id, name: user.name, role: user.role } });
});

// Test endpoint (protected)
app.get("/api/profile", authenticate, (req, res) => {
  res.json({ message: "Profil verisi", user: req.user });
});

app.listen(PORT, () => {
  console.log(`Sunucu ${PORT} portunda çalışıyor`);
});


// 📁 frontend/src/App.jsx
import React from "react";
import { useState, useEffect } from "react";

function App() {
  const [message, setMessage] = useState("");

  useEffect(() => {
    fetch("http://localhost:5000/")
      .then((res) => res.text())
      .then(setMessage);
  }, []);

  return (
    <div className="flex items-center justify-center min-h-screen bg-blue-100">
      <h1 className="text-2xl font-bold text-blue-700">{message}</h1>
    </div>
  );
}

export default App;


// 📁 frontend/src/main.jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App.jsx';
import './index.css';

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);


// 📁 frontend/index.html (Vite template)
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>EduNet</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>


// 📄 .env (backend klasörüne)
MONGO_URI=mongodb+srv://kullanici:parola@cluster.mongodb.net/edunet
PORT=5000
JWT_SECRET=supersecretkey123
