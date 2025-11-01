server.js
const express = require('express');
const mongoose = require('mongoose');
const path = require('path');
const app = express();
app.use(express.json());
app.use(express.static(__dirname)); // to serve index.html
mongoose.connect('mongodb://localhost:27017/studentdb')
.then(() => console.log('MongoDB Connected'))
.catch(err => console.log(err));
const Student = mongoose.model('Student', new mongoose.Schema({
name: String,
age: Number,
dept: String
}));
// CREATE
app.post('/add', async (req, res) => {
const s = await Student.create(req.body);
res.send(s);
});
// READ
app.get('/students', async (req, res) => {
const data = await Student.find();
res.send(data);
});
// UPDATE
app.put('/update/:id', async (req, res) => {
const s = await Student.findByIdAndUpdate(req.params.id, req.body, { new: true });
res.send(s);
});
// DELETE
app.delete('/delete/:id', async (req, res) => {
const s = await Student.findByIdAndDelete(req.params.id);
res.send(s);
});
app.get('/', (req, res) => {
res.sendFile(path.join(__dirname, 'index.html'));
});
app.listen(3000, () => console.log('Server running at http://localhost:3000'));index.html
<!DOCTYPE html>
<html>
<head>
<title>Student CRUD</title>
<script>
let editId = null;
async function addOrUpdate() {
const data = {
name: document.getElementById('name').value,
age: document.getElementById('age').value,
dept: document.getElementById('dept').value
};
if (editId) {
// UPDATE
await fetch('/update/' + editId, {
method: 'PUT',
headers: {'Content-Type': 'application/json'},
body: JSON.stringify(data)
});
editId = null;
} else {
// ADD
await fetch('/add', {
method: 'POST',
headers: {'Content-Type': 'application/json'},
body: JSON.stringify(data)
});
}
document.getElementById('name').value = '';
document.getElementById('age').value = '';
document.getElementById('dept').value = '';
document.getElementById('btn').innerText = 'Add';
getStudents();
}
async function getStudents() {
const res = await fetch('/students');
const students = await res.json();
document.getElementById('list').innerHTML = students.map(s => `
<li>
${s.name} (${s.age}, ${s.dept})
<button onclick="edit('${s._id}','${s.name}',${s.age},'${s.dept}')">✏️ Edit</button>
<button onclick="del('${s._id}')">❌ Delete</button>
</li>`).join('');
}
async function del(id) {
await fetch('/delete/' + id, { method: 'DELETE' });
getStudents();
}
function edit(id, name, age, dept) {
editId = id;
document.getElementById('name').value = name;
document.getElementById('age').value = age;
document.getElementById('dept').value = dept;
document.getElementById('btn').innerText = 'Update';
}
window.onload = getStudents;
</script>
</head>
<body style="font-family:Arial;text-align:center;">
<h2>Student Information</h2>
<input id="name" placeholder="Name">
<input id="age" type="number" placeholder="Age">
<input id="dept" placeholder="Dept">
<button id="btn" onclick="addOrUpdate()">Add</button>
<h3>Student List</h3>
<ul id="list"></ul>
</body>
</html>
HOW TO RUN
mkdir student-crud
cd student-crud
npm init –y
npm install express mangoose cors
