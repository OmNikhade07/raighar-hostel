<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Library Control System - Admin</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f4f6f8;
      padding: 20px;
      color: #333;
    }

    h1 {
      text-align: center;
      color: #2c3e50;
    }

    .container {
      max-width: 1000px;
      margin: auto;
      background: #fff;
      padding: 20px;
      border-radius: 10px;
      box-shadow: 0 4px 8px rgba(0,0,0,0.1);
    }

    form {
      margin-bottom: 30px;
    }

    input, select, button {
      padding: 10px;
      margin: 5px;
      width: 90%;
      max-width: 300px;
      border: 1px solid #ccc;
      border-radius: 8px;
      display: block;
    }

    button {
      background-color: #3498db;
      color: white;
      cursor: pointer;
      border: none;
      transition: 0.3s;
    }

    button:hover {
      background-color: #2980b9;
    }

    .section {
      margin-bottom: 40px;
    }

    table {
      width: 100%;
      border-collapse: collapse;
      margin-top: 10px;
    }

    th, td {
      padding: 10px;
      border-bottom: 1px solid #ddd;
      text-align: left;
    }

    th {
      background-color: #f0f0f0;
    }

    .issued {
      background-color: #dff0d8;
    }

    .returned {
      background-color: #f2dede;
    }

    .hidden {
      display: none;
    }

    .logout-btn {
      background: crimson;
      margin: 10px;
      width: 120px;
    }
  </style>
</head>
<body>
  <h1>📚 Library Control System</h1>
  <div class="container">

    <!-- Login Section -->
    <div id="loginSection">
      <h2>Admin Login</h2>
      <form id="loginForm">
        <input type="text" id="username" placeholder="Username" required>
        <input type="password" id="password" placeholder="Password" required>
        <button type="submit">Login</button>
      </form>
    </div>

    <!-- Main App Section -->
    <div id="appSection" class="hidden">
      <button class="logout-btn" onclick="logout()">Logout</button>

      <!-- Add Book -->
      <div class="section">
        <h2>Add Book</h2>
        <form id="addBookForm">
          <input type="text" id="bookTitle" placeholder="Book Title" required>
          <input type="text" id="authorName" placeholder="Author Name" required>
          <button type="submit">Add Book</button>
        </form>
      </div>

      <!-- Issue/Return Book -->
      <div class="section">
        <h2>Issue / Return Book</h2>
        <form id="issueReturnForm">
          <select id="bookSelect" required>
            <option value="">-- Select Book --</option>
          </select>
          <select id="action" required>
            <option value="issue">📤 Issue</option>
            <option value="return">📥 Return</option>
          </select>
          <input type="text" id="studentName" placeholder="Student Name" required>
          <button type="submit">Submit</button>
        </form>
      </div>

      <!-- Log Table -->
      <div class="section">
        <h2>Book Activity Log</h2>
        <table id="logTable">
          <thead>
            <tr>
              <th>Book Title</th>
              <th>Author</th>
              <th>Student</th>
              <th>Action</th>
              <th>Date</th>
            </tr>
          </thead>
          <tbody>
            <!-- Logs will be inserted here -->
          </tbody>
        </table>
      </div>
    </div>
  </div>

  <script>
    const ADMIN_USERNAME = "admin";
    const ADMIN_PASSWORD = "admin123";

    const books = JSON.parse(localStorage.getItem("books") || "[]");
    const log = JSON.parse(localStorage.getItem("log") || "[]");

    const loginSection = document.getElementById("loginSection");
    const appSection = document.getElementById("appSection");
    const bookSelect = document.getElementById("bookSelect");
    const logTableBody = document.querySelector("#logTable tbody");

    function updateBookDropdown() {
      bookSelect.innerHTML = '<option value="">-- Select Book --</option>';
      books.forEach((book, index) => {
        bookSelect.innerHTML += `<option value="${index}">${book.title} by ${book.author}</option>`;
      });
    }

    function updateLogTable() {
      logTableBody.innerHTML = "";
      log.slice().reverse().forEach(entry => {
        const tr = document.createElement("tr");
        tr.className = entry.action === "issue" ? "issued" : "returned";
        tr.innerHTML = `
          <td>${entry.title}</td>
          <td>${entry.author}</td>
          <td>${entry.student}</td>
          <td>${entry.action.toUpperCase()}</td>
          <td>${entry.date}</td>
        `;
        logTableBody.appendChild(tr);
      });
    }

    // Add Book
    document.getElementById("addBookForm").addEventListener("submit", function(e) {
      e.preventDefault();
      const title = document.getElementById("bookTitle").value.trim();
      const author = document.getElementById("authorName").value.trim();
      books.push({ title, author });
      localStorage.setItem("books", JSON.stringify(books));
      updateBookDropdown();
      this.reset();
      alert("Book added!");
    });

    // Issue/Return
    document.getElementById("issueReturnForm").addEventListener("submit", function(e) {
      e.preventDefault();
      const bookIndex = bookSelect.value;
      const student = document.getElementById("studentName").value.trim();
      const action = document.getElementById("action").value;
      const book = books[bookIndex];

      if (!book) return alert("Select a valid book!");

      log.push({
        title: book.title,
        author: book.author,
        student,
        action,
        date: new Date().toLocaleString()
      });

      localStorage.setItem("log", JSON.stringify(log));
      updateLogTable();
      this.reset();
    });

    // Admin Login
    document.getElementById("loginForm").addEventListener("submit", function(e) {
      e.preventDefault();
      const user = document.getElementById("username").value;
      const pass = document.getElementById("password").value;

      if (user === ADMIN_USERNAME && pass === ADMIN_PASSWORD) {
        localStorage.setItem("adminLoggedIn", "true");
        showApp();
      } else {
        alert("Invalid credentials!");
      }
    });

    function showApp() {
      loginSection.classList.add("hidden");
      appSection.classList.remove("hidden");
      updateBookDropdown();
      updateLogTable();
    }

    function logout() {
      localStorage.removeItem("adminLoggedIn");
      location.reload();
    }

    // Auto login if already logged in
    if (localStorage.getItem("adminLoggedIn") === "true") {
      showApp();
    }
  </script>
</body>
</html>

