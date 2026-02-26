<script>

// ================= PASSWORD DATABASE =================
const teacherPasswords = {
 "MsVictoria":"vic123",
 "MsRita":"rita123",
 "MsJudith":"jud123",
 "MsIfeoluwa":"ife123",
 "MsZainab":"zai123",
 "MsMary":"mary123",
 "MsMaryjane":"mj123",
 "MrJacob":"jac123",
 "MsAyoola":"ayo123",
 "MsImaobong":"ima123"
};

const adminPassword = "admin123";

let userRole = "";
let teacherId = "";
let selectedTime = "";

let bookings = JSON.parse(localStorage.getItem("fairviewBookings")) || [];

// ================= LOGIN =================
function login(){
 const role = document.getElementById("role").value;
 const pass = document.getElementById("password").value;

 if(role === "teacher"){
   const teacher = prompt("Enter Teacher Name exactly (e.g MsVictoria)");
   if(teacherPasswords[teacher] !== pass){
      alert("Invalid Teacher Details");
      return;
   }
   teacherId = teacher;
   userRole = "teacher";
 }

 if(role === "admin"){
   if(pass !== adminPassword){
      alert("Invalid Admin Password");
      return;
   }
   userRole = "admin";
 }

 if(role === "parent"){
   userRole = "parent";
 }

 document.getElementById("loginSection").classList.add("hidden");

 if(userRole === "parent"){
   document.getElementById("bookingSection").classList.remove("hidden");
   document.getElementById("adminSection").classList.remove("hidden");
   loadClasses();
   loadDashboard();
 }

 if(userRole === "teacher" || userRole === "admin"){
   document.getElementById("adminSection").classList.remove("hidden");
   loadDashboard();
 }
}

// ================= CLASSES =================
const classes = [
"Year 1 Onyx","Year 1 Amber","Year 2 Ruby","Year 2 Beryl",
"Year 3 Crystal","Year 3 Silver","Year 4 Gold","Year 4 Topaz",
"Year 5 Diamond","Year 5 Opal","Year 6 Pearl"
];

function loadClasses(){
 let select = document.getElementById("classSelect");
 select.innerHTML="";
 classes.forEach(c=>{
   let opt=document.createElement("option");
   opt.text=c;
   select.add(opt);
 });
 generateSlots();
}

// ================= SLOT GENERATION =================
function generateSlots(){
 let container=document.getElementById("slots");
 container.innerHTML="";

 for(let hour=9; hour<15; hour++){
  for(let min=0; min<60; min+=15){

    let time = String(hour).padStart(2,'0')+":"+String(min).padStart(2,'0');

    if(time==="12:30" || time==="12:45"){
      let div=document.createElement("div");
      div.className="slot disabled";
      div.innerHTML=time+" (Break)";
      container.appendChild(div);
      continue;
    }

    let div=document.createElement("div");
    div.className="slot available";
    div.innerHTML=time;
    div.onclick=()=>selectSlot(time, div);
    container.appendChild(div);
  }
 }
 refreshSlots();
}

// ================= SELECT SLOT =================
function selectSlot(time, element){
 selectedTime = time;

 document.querySelectorAll(".slot").forEach(s=>{
   s.style.border="none";
 });

 element.style.border="3px solid gold";
}

// ================= BOOK SLOT =================
function submitBooking(){
 let parent=document.getElementById("parentName").value;
 let phone=document.getElementById("phone").value;
 let teacher=document.getElementById("teacherSelect").value;
 let className=document.getElementById("classSelect").value;

 if(!parent || !phone || !teacher || !selectedTime){
  alert("Fill all fields and select time");
  return;
 }

 if(bookings.find(b=>b.className===className && b.time===selectedTime)){
  alert("Slot already booked");
  return;
 }

 bookings.push({className,time:selectedTime,parent,phone,teacher});
 localStorage.setItem("fairviewBookings",JSON.stringify(bookings));

 selectedTime="";
 alert("Booking Confirmed");

 generateSlots();
 loadDashboard();
}

// ================= REFRESH SLOT =================
function refreshSlots(){
 let className=document.getElementById("classSelect").value;

 document.querySelectorAll(".slot").forEach(s=>{
  let time=s.innerText.replace(" (Break)","");
  let booked = bookings.find(b=>b.className===className && b.time===time);

  if(booked){
    s.className="slot booked";
    s.innerHTML=time+"<br><small>"+booked.parent+"</small>";
  }
 });
}

// ================= DASHBOARD =================
function loadDashboard(){
 let table=document.getElementById("bookingTable");
 table.innerHTML="<tr><th>Class</th><th>Time</th><th>Parent</th><th>Phone</th><th>Teacher</th></tr>";

 document.getElementById("dashboardTitle").innerText =
   (userRole==="admin") ? "Admin Dashboard" :
   (userRole==="teacher") ? "Teacher Dashboard" :
   "All Bookings";

 bookings
  .filter(b => userRole==="admin" || userRole==="parent" || b.teacher===teacherId)
  .forEach(b=>{
    table.innerHTML+=`
    <tr>
      <td>${b.className}</td>
      <td>${b.time}</td>
      <td>${b.parent}</td>
      <td>${b.phone}</td>
      <td>${b.teacher}</td>
    </tr>`;
  });

 if(userRole==="admin"){
   table.innerHTML += `
   <tr>
     <td colspan="5">
       <button onclick="exportCSV()">Export to Excel</button>
       <button onclick="clearAll()">Clear All Bookings</button>
       <button onclick="location.reload()">Logout</button>
     </td>
   </tr>`;
 }

 if(userRole==="teacher" || userRole==="parent"){
   table.innerHTML += `
   <tr>
     <td colspan="5">
       <button onclick="location.reload()">Logout</button>
     </td>
   </tr>`;
 }
}

// ================= EXPORT =================
function exportCSV(){
 let csv = "Class,Time,Parent,Phone,Teacher\n";
 bookings.forEach(b=>{
   csv+=`${b.className},${b.time},${b.parent},${b.phone},${b.teacher}\n`;
 });

 let blob = new Blob([csv],{type:"text/csv"});
 let url = URL.createObjectURL(blob);
 let a=document.createElement("a");
 a.href=url;
 a.download="Fairview_PTC_Bookings.csv";
 a.click();
}

// ================= CLEAR =================
function clearAll(){
 if(confirm("Are you sure?")){
   bookings=[];
   localStorage.removeItem("fairviewBookings");
   loadDashboard();
   generateSlots();
 }
}

</script>
