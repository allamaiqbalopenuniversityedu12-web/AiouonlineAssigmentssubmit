# AiouonlineAssigmentssubmit
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>AIOU Assignment Portal with PDF</title>
<style>
  body { font-family: Arial; background: #f0f0f0; text-align: center; padding: 50px; }
  .login-box, .dashboard { background: white; padding: 20px; border-radius: 10px; width: 500px; margin: auto; margin-top: 50px; }
  input, select, button { width: 90%; padding: 10px; margin: 10px 0; }
  table { width: 100%; border-collapse: collapse; margin-top: 20px; }
  th, td { border: 1px solid #ddd; padding: 8px; }
  th { background-color: #4CAF50; color: white; }
  .btn { padding: 5px 10px; margin: 2px; cursor: pointer; }
  .accept { background-color: #4CAF50; color: white; }
  .reject { background-color: #f44336; color: white; }
  .download { background-color: #2196F3; color: white; }
</style>
</head>
<body>
<h1>AIOU Assignment Portal</h1>

<div class="login-box" id="loginBox">
  <h2>Login</h2>
  <select id="role">
    <option value="student">Student</option>
    <option value="admin">Admin</option>
  </select><br>
  <input type="text" id="username" placeholder="Username / CNIC"><br>
  <input type="password" id="password" placeholder="Password"><br>
  <button onclick="login()">Login</button>
</div>

<!-- Admin Dashboard -->
<div class="dashboard" id="adminDashboard" style="display:none;">
  <h2>Admin Dashboard</h2>
  <p>Welcome, Admin!</p>
  <table id="assignmentTable">
    <tr>
      <th>Student Name</th>
      <th>Course</th>
      <th>Assignment File</th>
      <th>Status</th>
      <th>Action</th>
    </tr>
  </table>
  <button onclick="logout()">Logout</button>
</div>

<!-- Student Dashboard -->
<div class="dashboard" id="studentDashboard" style="display:none;">
  <h2>Student Dashboard</h2>
  <p>Welcome, Student!</p>
  <input type="text" id="studentName" placeholder="Enter Your Name"><br>
  <input type="text" id="studentCourse" placeholder="Enter Course Code"><br>
  <input type="file" id="assignmentFile" accept=".pdf"><br>
  <button onclick="uploadAssignment()">Upload Assignment</button>

  <h3>Your Assignments</h3>
  <table id="studentTable">
    <tr>
      <th>Assignment File</th>
      <th>Status</th>
      <th>Download</th>
    </tr>
  </table>

  <button onclick="logout()">Logout</button>
</div>

<script>
let assignments = [];

function login() {
  const role = document.getElementById('role').value;
  const username = document.getElementById('username').value;
  const password = document.getElementById('password').value;

  if(role === 'admin' && username === 'admin' && password === 'admin123') {
    document.getElementById('loginBox').style.display = 'none';
    document.getElementById('adminDashboard').style.display = 'block';
    loadAssignments();
  } 
  else if(role === 'student' && username === 'student' && password === 'student123') {
    document.getElementById('loginBox').style.display = 'none';
    document.getElementById('studentDashboard').style.display = 'block';
    loadStudentAssignments();
  } 
  else {
    alert('Invalid credentials!');
  }
}

function uploadAssignment() {
  const name = document.getElementById('studentName').value.trim();
  const course = document.getElementById('studentCourse').value.trim();
  const fileInput = document.getElementById('assignmentFile');
  if(!name || !course || fileInput.files.length === 0) {
    alert('Please enter name, course and select a PDF!');
    return;
  }
  const file = fileInput.files[0];
  const reader = new FileReader();
  reader.onload = function(e) {
    assignments.push({
      name, 
      course, 
      fileName: file.name, 
      fileData: e.target.result, // store PDF base64
      status: 'Pending'
    });
    alert('Assignment uploaded successfully!');
    fileInput.value = '';
    document.getElementById('studentName').value = '';
    document.getElementById('studentCourse').value = '';
    loadStudentAssignments();
    loadAssignments();
  }
  reader.readAsDataURL(file);
}

function loadStudentAssignments() {
  const table = document.getElementById('studentTable');
  table.innerHTML = `<tr>
    <th>Assignment File</th>
    <th>Status</th>
    <th>Download</th>
  </tr>`;
  assignments.forEach((a, index) => {
    const row = table.insertRow();
    row.insertCell(0).innerText = a.fileName;
    row.insertCell(1).innerText = a.status;
    const downloadCell = row.insertCell(2);
    const dlBtn = document.createElement("button");
    dlBtn.className = "btn download";
    dlBtn.innerText = "Download";
    dlBtn.onclick = () => downloadPDF(a.fileName, a.fileData);
    downloadCell.appendChild(dlBtn);
  });
}

function loadAssignments() {
  const table = document.getElementById('assignmentTable');
  table.innerHTML = `<tr>
    <th>Student Name</th>
    <th>Course</th>
    <th>Assignment File</th>
    <th>Status</th>
    <th>Action</th>
  </tr>`;
  assignments.forEach((a, index) => {
    const row = table.insertRow();
    row.insertCell(0).innerText = a.name;
    row.insertCell(1).innerText = a.course;
    row.insertCell(2).innerText = a.fileName;
    row.insertCell(3).innerText = a.status;
    const actionCell = row.insertCell(4);
    if(a.status === "Pending") {
      const acceptBtn = document.createElement("button");
      acceptBtn.className = "btn accept";
      acceptBtn.innerText = "Accept";
      acceptBtn.onclick = () => updateStatus(index, "Accepted");
      const rejectBtn = document.createElement("button");
      rejectBtn.className = "btn reject";
      rejectBtn.innerText = "Reject";
      rejectBtn.onclick = () => updateStatus(index, "Rejected");
      const dlBtn = document.createElement("button");
      dlBtn.className = "btn download";
      dlBtn.innerText = "Download";
      dlBtn.onclick = () => downloadPDF(a.fileName, a.fileData);
      actionCell.appendChild(acceptBtn);
      actionCell.appendChild(rejectBtn);
      actionCell.appendChild(dlBtn);
    } else {
      const dlBtn = document.createElement("button");
      dlBtn.className = "btn download";
      dlBtn.innerText = "Download";
      dlBtn.onclick = () => downloadPDF(a.fileName, a.fileData);
      actionCell.appendChild(dlBtn);
    }
  });
}

function updateStatus(index, status) {
  assignments[index].status = status;
  loadAssignments();
  loadStudentAssignments();
}

function downloadPDF(fileName, fileData) {
  const a = document.createElement("a");
  a.href = fileData;
  a.download = fileName;
  a.click();
}

function logout() {
  document.getElementById('loginBox').style.display = 'block';
  document.getElementById('adminDashboard').style.display = 'none';
  document.getElementById('studentDashboard').style.display = 'none';
  document.getElementById('username').value = '';
  document.getElementById('password').value = '';
}
</script>
</body>
</html>
