We already built a **very good backend** (Students, Attendance, Performance, Auth). 👍
Your HTML only shows **Students**, so we just need to extend the dashboard to manage:

1. **Students**
2. **Attendance**
3. **Performance**

Here's a **clean simple UI with 3 sections** so you can add and view everything.

No extra backend changes needed — it uses your existing APIs.

---

# Complete Updated HTML + CSS + JS

Replace your current HTML with this.

```html
<!DOCTYPE html>
<html>
<head>
<title>EduTrack System</title>

<style>

body{
font-family: Arial;
margin:0;
height:100vh;
display:flex;
justify-content:center;
align-items:center;
background:linear-gradient(135deg,#ffd6ec,#e5d4ff);
}

.container{
background:white;
padding:30px;
width:800px;
border-radius:15px;
box-shadow:0 10px 25px rgba(0,0,0,0.15);
text-align:center;
}

h2{
color:#8a2be2;
margin-bottom:20px;
}

input{
padding:8px;
margin:6px;
border-radius:8px;
border:2px solid #e0c8ff;
}

button{
padding:8px 15px;
margin:5px;
border:none;
border-radius:8px;
color:white;
cursor:pointer;
}

.signup{
background:linear-gradient(135deg,#ff9bd6,#b388ff);
}

.login{
background:linear-gradient(135deg,#b388ff,#8a2be2);
}

.section{
margin-top:25px;
padding:10px;
border-top:2px solid #eee;
}

table{
width:100%;
border-collapse:collapse;
margin-top:10px;
}

th{
background:#b388ff;
color:white;
padding:8px;
}

td{
padding:6px;
border:1px solid #eee;
}

#dashboard{
display:none;
}

</style>
</head>

<body>

<div class="container">

<!-- LOGIN -->
<div id="authBox">

<h2>EduTrack Login</h2>

<input type="email" id="email" placeholder="Email"><br>
<input type="password" id="password" placeholder="Password"><br>

<button class="signup" onclick="signup()">Sign Up</button>
<button class="login" onclick="login()">Login</button>

<p id="msg"></p>

</div>

<!-- DASHBOARD -->
<div id="dashboard">

<h2>EduTrack Dashboard</h2>

<!-- STUDENTS -->
<div class="section">

<h3>Students</h3>

<table id="studentTable">
<tr>
<th>ID</th>
<th>Name</th>
<th>Course</th>
</tr>
</table>

<input id="studentName" placeholder="Student Name">
<input id="studentCourse" placeholder="Course">

<button class="signup" onclick="addStudent()">Add Student</button>

</div>

<!-- ATTENDANCE -->
<div class="section">

<h3>Attendance</h3>

<table id="attendanceTable">
<tr>
<th>ID</th>
<th>Student ID</th>
<th>Date</th>
<th>Status</th>
</tr>
</table>

<input id="attStudentId" placeholder="Student ID">
<input id="attDate" type="date">
<input id="attStatus" placeholder="Present/Absent">

<button class="signup" onclick="addAttendance()">Add Attendance</button>

</div>

<!-- PERFORMANCE -->
<div class="section">

<h3>Performance</h3>

<table id="performanceTable">
<tr>
<th>ID</th>
<th>Student ID</th>
<th>Subject</th>
<th>Marks</th>
</tr>
</table>

<input id="perfStudentId" placeholder="Student ID">
<input id="perfSubject" placeholder="Subject">
<input id="perfMarks" placeholder="Marks">

<button class="signup" onclick="addPerformance()">Add Performance</button>

</div>

</div>

</div>


<script>

function signup(){

let email=document.getElementById("email").value
let password=document.getElementById("password").value

fetch("http://localhost:8080/auth/signup",{

method:"POST",
headers:{"Content-Type":"application/json"},
body:JSON.stringify({email:email,password:password})

})

.then(res=>res.text())
.then(msg=>document.getElementById("msg").innerText=msg)

}



function login(){

let email=document.getElementById("email").value
let password=document.getElementById("password").value

fetch("http://localhost:8080/auth/signin",{

method:"POST",
headers:{"Content-Type":"application/json"},
body:JSON.stringify({email:email,password:password})

})

.then(res=>res.text())

.then(msg=>{

document.getElementById("msg").innerText=msg

if(msg==="Login successful!"){

document.getElementById("authBox").style.display="none"
document.getElementById("dashboard").style.display="block"

loadStudents()
loadAttendance()
loadPerformance()

}

})

}


function loadStudents(){

fetch("http://localhost:8080/students")

.then(res=>res.json())

.then(data=>{

let table=document.getElementById("studentTable")

table.innerHTML="<tr><th>ID</th><th>Name</th><th>Course</th></tr>"

data.forEach(s=>{

let row=table.insertRow()

row.insertCell(0).innerText=s.id
row.insertCell(1).innerText=s.name
row.insertCell(2).innerText=s.course

})

})

}


function addStudent(){

let name=document.getElementById("studentName").value
let course=document.getElementById("studentCourse").value

fetch("http://localhost:8080/students",{

method:"POST",
headers:{"Content-Type":"application/json"},
body:JSON.stringify({name:name,course:course})

})

.then(()=>loadStudents())

}



function loadAttendance(){

fetch("http://localhost:8080/attendance/all")

.then(res=>res.json())

.then(data=>{

let table=document.getElementById("attendanceTable")

table.innerHTML="<tr><th>ID</th><th>Student ID</th><th>Date</th><th>Status</th></tr>"

data.forEach(a=>{

let row=table.insertRow()

row.insertCell(0).innerText=a.id
row.insertCell(1).innerText=a.studentId
row.insertCell(2).innerText=a.date
row.insertCell(3).innerText=a.status

})

})

}



function addAttendance(){

let studentId=document.getElementById("attStudentId").value
let date=document.getElementById("attDate").value
let status=document.getElementById("attStatus").value

fetch("http://localhost:8080/attendance/add",{

method:"POST",
headers:{"Content-Type":"application/json"},
body:JSON.stringify({
studentId:studentId,
date:date,
status:status
})

})

.then(()=>loadAttendance())

}



function loadPerformance(){

fetch("http://localhost:8080/performance")

.then(res=>res.json())

.then(data=>{

let table=document.getElementById("performanceTable")

table.innerHTML="<tr><th>ID</th><th>Student ID</th><th>Subject</th><th>Marks</th></tr>"

data.forEach(p=>{

let row=table.insertRow()

row.insertCell(0).innerText=p.id
row.insertCell(1).innerText=p.studentId
row.insertCell(2).innerText=p.subject
row.insertCell(3).innerText=p.marks

})

})

}



function addPerformance(){

let studentId=document.getElementById("perfStudentId").value
let subject=document.getElementById("perfSubject").value
let marks=document.getElementById("perfMarks").value

fetch("http://localhost:8080/performance",{

method:"POST",
headers:{"Content-Type":"application/json"},
body:JSON.stringify({
studentId:studentId,
subject:subject,
marks:marks
})

})

.then(()=>loadPerformance())

}

</script>

</body>
</html>
```

---

# What this UI can do now

After login you can:

### Students

* View students
* Add student

### Attendance

* Add attendance
* View attendance

### Performance

* Add marks
* View marks

---

# Example Data Flow

Add Student

```
Name: Rahul
Course: CSE
```

Add Attendance

```
Student ID: 1
Date: 2026-03-10
Status: Present
```

Add Performance

```
Student ID: 1
Subject: Java
Marks: 85
```
# Screenshot 
<img width="1912" height="876" alt="image" src="https://github.com/user-attachments/assets/5d3716e6-2e29-4b37-af7e-09214bdb3a9c" />

---

# Small Backend Issue You Must Fix

Your **Student.java**

```
@Id
private int id;
```

You should add auto increment:

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private int id;
```

Otherwise student insertion may fail.


