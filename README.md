# raighar-hostel
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Raighar Boys Hostel</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #eef1f4;
      padding: 20px;
    }
    h1, h2 {
      color: #2c3e50;
    }
    .section {
      background: white;
      padding: 15px;
      margin-bottom: 20px;
      border-radius: 10px;
      box-shadow: 0 2px 5px rgba(0,0,0,0.1);
    }
    input, select, button {
      margin-top: 5px;
      padding: 8px;
      width: 100%;
      margin-bottom: 10px;
    }
    ul {
      padding-left: 20px;
    }
    table {
      width: 100%;
      border-collapse: collapse;
    }
    th, td {
      padding: 8px;
      border: 1px solid #ccc;
    }
    th {
      background: #dff0ff;
    }
    #loginBox {
      max-width: 400px;
      margin: auto;
      display: none;
    }
    .hidden {
      display: none;
    }
    img {
      max-height: 40px;
      border-radius: 5px;
      margin-left: 5px;
      vertical-align: middle;
    }
    #logoutBtn {
      float: right;
      background: red;
      color: white;
      width: auto;
      padding: 5px 10px;
      border-radius: 5px;
    }
    @media screen and (max-width: 600px) {
      body { padding: 10px; }
      h1 { font-size: 22px; }
      .section { padding: 10px; }
    }
  </style>
</head>
<body>

<button onclick="showLogin()" style="float:right;margin-bottom:10px;">Login</button>
<button id="logoutBtn" class="hidden" onclick="logout()">Logout</button>

<h1>Raighar Boys Hostel Dashboard</h1>

<div class="section">
  <h2>Hostel Management</h2>
  <ul>
    <li><strong>Rector:</strong> Dr. Y. Bisen</li>
    <li><strong>Warden:</strong> Shree Dilip Nikhade</li>
    <li><strong>Security:</strong> Shree Shiva Bhau</li>
    <li><strong>Workers:</strong> Ram Bhau, Pawar Bhau, Ritesh Bhau</li>
    <li><strong>Current Prefect:</strong> Saurabh Goverdipe</li>
  </ul>
  <p><strong>Login Role:</strong> <span id="roleDisplay">None (View-only)</span></p>
</div>

<!-- Login Box -->
<div class="section" id="loginBox">
  <h2>Login</h2>
  <input type="text" id="username" placeholder="Username">
  <input type="password" id="password" placeholder="Password">
  <button onclick="login()">Login</button>
  <p id="loginMessage" style="color:red;"></p>
</div>

<!-- Committee Section -->
<div class="section">
  <h2>Committees</h2>
  <div id="committeeControls" class="hidden">
    <label>Select Committee</label>
    <select id="committeeName">
      <option value="Mess Committee">Mess Committee</option>
      <option value="Maintenance Committee">Maintenance Committee</option>
      <option value="Sports Committee">Sports Committee</option>
      <option value="Gardening Committee">Gardening Committee</option>
      <option value="Library Committee">Library Committee</option>
    </select>
    <input type="text" id="memberName" placeholder="Enter Member Name">
    <input type="file" id="studentImage" accept="image/*">
    <button onclick="addCommitteeMember()">Add Member</button>
    <button onclick="exportCommittees()">Export Committee Data (CSV)</button>
  </div>
  <div id="committeeList"></div>
</div>

<!-- Library Section -->
<div class="section">
  <h2>Library</h2>
  <div id="libraryControls" class="hidden">
    <input type="text" id="bookName" placeholder="Book Title">
    <input type="text" id="bookAuthor" placeholder="Author">
    <button onclick="addBook()">Add Book</button>
    <button onclick="exportLibrary()">Export Library Data (CSV)</button>
  </div>

  <h3>Available Books</h3>
  <table id="bookTable">
    <thead>
      <tr>
        <th>Title</th>
        <th>Author</th>
        <th>Status</th>
        <th>User</th>
        <th>Actions</th>
      </tr>
    </thead>
    <tbody></tbody>
  </table>
</div>

<script>
  let userRole = "";
  let committees = {};
  let libraryBooks = [];

  function showLogin() {
    document.getElementById("loginBox").style.display = "block";
  }

  function login() {
    const user = document.getElementById("username").value.trim().toLowerCase();
    const pass = document.getElementById("password").value.trim();
    const msg = document.getElementById("loginMessage");

    if (user === "admin" && pass === "admin123") {
      userRole = "Admin";
    } else if (user === "prefect" && pass === "saurabh123") {
      userRole = "Prefect";
    } else if (user === "rector" && pass === "bisen123") {
      userRole = "Rector";
    } else {
      msg.textContent = "Invalid credentials!";
      return;
    }

    document.getElementById("loginBox").style.display = "none";
    document.getElementById("roleDisplay").textContent = userRole;
    document.getElementById("logoutBtn").classList.remove("hidden");

    if (userRole === "Admin" || userRole === "Prefect") {
      document.getElementById("committeeControls").classList.remove("hidden");
      document.getElementById("libraryControls").classList.remove("hidden");
    }

    displayCommittees();
    displayBooks();
  }

  function logout() {
    userRole = "";
    document.getElementById("roleDisplay").textContent = "None (View-only)";
    document.getElementById("committeeControls").classList.add("hidden");
    document.getElementById("libraryControls").classList.add("hidden");
    document.getElementById("logoutBtn").classList.add("hidden");
    displayCommittees();
    displayBooks();
  }

  function saveCommittees() {
    localStorage.setItem("committees", JSON.stringify(committees));
  }

  function loadCommittees() {
    const data = localStorage.getItem("committees");
    committees = data ? JSON.parse(data) : {};
  }

  function addCommitteeMember() {
    const committee = document.getElementById("committeeName").value;
    const member = document.getElementById("memberName").value;
    const fileInput = document.getElementById("studentImage");

    if (!committees[committee]) committees[committee] = [];

    if (member) {
      if (fileInput.files.length > 0) {
        const reader = new FileReader();
        reader.onload = function(e) {
          const imageData = e.target.result;
          committees[committee].push(`${member} <img src="${imageData}" width="40">`);
          displayCommittees();
          saveCommittees();
        };
        reader.readAsDataURL(fileInput.files[0]);
      } else {
        committees[committee].push(member);
        displayCommittees();
        saveCommittees();
      }
      document.getElementById("memberName").value = '';
      fileInput.value = '';
    }
  }

  function removeCommitteeMember(committee, index) {
    if (committees[committee]) {
      committees[committee].splice(index, 1);
      if (committees[committee].length === 0) delete committees[committee];
      saveCommittees();
      displayCommittees();
    }
  }

  function displayCommittees() {
    const div = document.getElementById("committeeList");
    div.innerHTML = "";
    for (let committee in committees) {
      const members = committees[committee];
      const list = members.map((member, index) => {
        return `<li>
          ${member}
          ${userRole === "Admin" || userRole === "Prefect" ? 
            `<button onclick="removeCommitteeMember('${committee}', ${index})" style="margin-left:10px;">Remove</button>` : ""}
        </li>`;
      }).join("");
      div.innerHTML += `<div>
        <strong>${committee}:</strong>
        <ul>${list}</ul>
      </div>`;
    }
  }

  function saveBooks() {
    localStorage.setItem("libraryBooks", JSON.stringify(libraryBooks));
  }

  function loadBooks() {
    const data = localStorage.getItem("libraryBooks");
    libraryBooks = data ? JSON.parse(data) : [];
  }

  function addBook() {
    const title = document.getElementById("bookName").value;
    const author = document.getElementById("bookAuthor").value;
    if (title && author) {
      libraryBooks.push({ title, author, status: "Available", user: "-", date: null });
      displayBooks();
      saveBooks();
      document.getElementById("bookName").value = '';
      document.getElementById("bookAuthor").value = '';
    }
  }

  function removeBook(index) {
    if (confirm("Are you sure you want to delete this book?")) {
      libraryBooks.splice(index, 1);
      saveBooks();
      displayBooks();
    }
  }

  function displayBooks() {
    const tbody = document.getElementById("bookTable").querySelector("tbody");
    tbody.innerHTML = "";
    const today = new Date();

    libraryBooks.forEach((book, index) => {
      const issueDate = book.date ? new Date(book.date) : null;
      let overdue = "";

      if (issueDate) {
        const diff = Math.floor((today - issueDate) / (1000 * 60 * 60 * 24));
        if (diff > 7) overdue = "<span style='color:red;'> Overdue!</span>";
      }

      const row = tbody.insertRow();
      row.innerHTML = `
        <td>${book.title}</td>
        <td>${book.author}</td>
        <td>${book.status}</td>
        <td>${book.user}${overdue}</td>
        <td>
          ${userRole === "Admin" || userRole === "Prefect" ? `
            <button onclick="issueBook(${index})">Issue</button>
            <button onclick="returnBook(${index})">Return</button>
            <button onclick="removeBook(${index})">Remove</button>` : `View Only`}
        </td>
      `;
    });
  }

  function issueBook(index) {
    if (userRole !== "Admin" && userRole !== "Prefect") return;
    const name = prompt("Enter student name:");
    if (name) {
      libraryBooks[index].status = "Issued";
      libraryBooks[index].user = name;
      libraryBooks[index].date = new Date().toISOString();
      displayBooks();
      saveBooks();
    }
  }

  function returnBook(index) {
    if (userRole !== "Admin" && userRole !== "Prefect") return;
    libraryBooks[index].status = "Available";
    libraryBooks[index].user = "-";
    libraryBooks[index].date = null;
    displayBooks();
    saveBooks();
  }

  function exportCommittees() {
    let csv = "Committee,Member\n";
    for (const committee in committees) {
      committees[committee].forEach(member => {
        const cleanText = member.replace(/<[^>]+>/g, '');
        csv += `"${committee}","${cleanText}"\n`;
      });
    }
    downloadCSV(csv, "committees.csv");
  }

  function exportLibrary() {
    let csv = "Title,Author,Status,User,IssueDate\n";
    libraryBooks.forEach(book => {
      csv += `"${book.title}","${book.author}","${book.status}","${book.user}","${book.date || '-'}"\n`;
    });
    downloadCSV(csv, "library_books.csv");
  }

  function downloadCSV(csv, filename) {
    const blob = new Blob([csv], { type: "text/csv" });
    const url = window.URL.createObjectURL(blob);
    const a = document.createElement("a");
    a.href = url;
    a.download = filename;
    a.click();
    window.URL.revokeObjectURL(url);
  }

  // Load saved data on startup
  loadCommittees();
  loadBooks();
  displayCommittees();
  displayBooks();
</script>

</body>
</html>
