<!DOCTYPE html>
<html lang="th">
<head>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta charset="UTF-8">
  <title>จัดตารางเรียน - คณะรัฐศาสตร์ รามคำแหง</title>
  <link href="https://fonts.googleapis.com/css2?family=Prompt:wght@300;400;600&display=swap" rel="stylesheet">
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html2pdf.js/0.10.1/html2pdf.bundle.min.js"></script>
  <link href="https://cdn.jsdelivr.net/npm/fullcalendar@6.1.8/main.min.css" rel="stylesheet">
  <script src="https://cdn.jsdelivr.net/npm/fullcalendar@6.1.8/main.min.js"></script>
  <style>
    body {
      margin: 0;
      font-family: 'Prompt', sans-serif;
      background-color: #f5f6fa;
      padding: 30px;
      text-align: center;
    }
    h1 { color: #1f2f56; margin-bottom: 0; }
    .university { font-size: 18px; color: #555; margin-bottom: 25px; }
    .container {
      background: white;
      padding: 24px;
      border-radius: 16px;
      box-shadow: 0 4px 12px rgba(0,0,0,0.08);
      max-width: 1200px;
      margin: auto;
    }
    input, select, button {
      padding: 10px;
      margin: 6px 4px;
      font-size: 16px;
      border-radius: 8px;
      border: 1px solid #ccc;
    }
    button {
      background-color: #1f2f56;
      color: white;
      border: none;
      cursor: pointer;
    }
    button:hover { background-color: #0d1b3f; }
    table.schedule {
      width: 100%;
      border-collapse: collapse;
      margin-top: 20px;
    }
    .schedule th, .schedule td {
      border: 1px solid #ddd;
      padding: 10px;
      text-align: center;
    }
    .schedule th {
      background-color: #1f2f56;
      color: white;
    }
    .class-box {
      border-radius: 10px;
      padding: 6px;
      font-size: 14px;
      color: white;
      cursor: pointer;
    }
    .edit-mode {
      background: white !important;
      color: black !important;
      border: 1px dashed #999;
    }
    .color1 { background-color: #f57c00; } /* orange */
    .color2 { background-color: #1976d2; } /* blue */
    .color3 { background-color: #d32f2f; } /* red */
    .color4 { background-color: #ffc107; color: black; } /* yellow */
  
  </style>
</head>
<body>
  <h1>📘 จัดตารางเรียน</h1>
  <div class="university">คณะรัฐศาสตร์ มหาวิทยาลัยรามคำแหง</div>

  <div class="container" id="capture">
    <input type="text" id="code" placeholder="รหัสวิชา">
    <input type="text" id="subject" placeholder="ชื่อวิชา">
    <input type="text" id="credit" placeholder="หน่วยกิต" inputmode="decimal">

    <select id="term">
      <option value="1">เทอม 1</option>
      <option value="2">เทอม 2</option>
    </select>
    <select id="year">
      <option value="1">ปี 1</option>
      <option value="2">ปี 2</option>
      <option value="3">ปี 3</option>
      <option value="4">ปี 4</option>
    </select>

    <label>วันที่:</label>
    <input type="date" id="date">
    <label>เวลาเริ่ม:</label>
    <input type="time" id="timeStart">
    <input type="time" id="timeEnd">
    <label>วันที่สอบ:</label>
    <input type="date" id="examDate">

    <button type="button" onclick="addSubject()">➕ เพิ่มวิชา</button>

    <table>
      <thead>
        <tr>
          <th>รหัสวิชา</th><th>ชื่อวิชา</th><th>หน่วยกิต</th><th>ปี</th><th>เทอม</th><th>วันที่</th><th>เวลาเริ่ม</th><th>เวลาเลิก</th><th>วันสอบ</th><th>ลบ</th>

        </tr>
      </thead>
      <tbody id="scheduleTable"></tbody>
    </table>

    <div class="summary">
      หน่วยกิตรวม: <span id="totalCredits">0</span>
    </div>
    <br>
    <button onclick="downloadPDF()">📄 ดาวน์โหลด PDF</button>
    <button onclick="showStudentView()">📋 ตารางแบบนักเรียน</button>
  </div>
  
  <div id="calendarView" style="display: none; margin-top: 20px;"></div>
  <div id="backButton" style="display: none; margin-top: 10px;">
    <button onclick="toggleCalendar()">🔙 ย้อนกลับไปตารางเรียน</button>
  </div>

  <div id="studentTableView" style="display: none; margin-top: 20px;">
    <button onclick="downloadStudentPDF()">📄 ดาวน์โหลด PDF (ตารางนักเรียน)</button>
    <h2>📄 ตารางเรียนแบบนักเรียน</h2>
    <table class="schedule">
      <thead>
        <tr>
          <th>วัน</th>
          <th>เวลา</th>
          <th>วิชา</th>
          <th>วันที่สอบ</th>
    
        </tr>
      </thead>
      <tbody id="studentViewBody">
        <!-- generateStudentView จะเติมข้อมูลตรงนี้ -->
      </tbody>
    </table>
    <button onclick="toggleStudentView()">🔙 ย้อนกลับ</button>
  </div>

  <!-- ✅ ย้าย script มาต่อท้าย -->
  <script>
    let subjects = [];
    let calendar;

    window.onload = () => {
      const saved = localStorage.getItem("schedule");
      if (saved) {
        subjects = JSON.parse(saved);
        subjects.forEach(addRow);
        updateSummary();
      }
    };

    function addSubject() {
  const code = document.getElementById("code").value.trim();
  const subject = document.getElementById("subject").value.trim();
  const creditInput = document.getElementById("credit").value.trim().replace(/[^\d.]/g, '');
  const credit = parseFloat(creditInput);
  const term = document.getElementById("term").value;
  const year = document.getElementById("year").value;
  const date = document.getElementById("date").value;
  const timeStart = document.getElementById("timeStart").value;
  const timeEnd = document.getElementById("timeEnd").value;
  const examDate = document.getElementById("examDate").value;

  if (!code || !subject || isNaN(credit) || credit <= 0 || !term || !year || !date || !timeStart || !timeEnd || !examDate) {
    window.alert("❗ กรุณากรอกข้อมูลให้ครบ โดยเฉพาะ 'หน่วยกิต' และ 'เวลา'");
    return;
  }

  const sub = { code, subject, credit, term, year, date, timeStart, timeEnd, examDate };
  subjects.push(sub);
  localStorage.setItem("schedule", JSON.stringify(subjects));
  addRow(sub);
  updateSummary();

  // reset form
  document.getElementById("code").value = "";
  document.getElementById("subject").value = "";
  document.getElementById("credit").value = "";
  document.getElementById("term").value = "1";
  document.getElementById("year").value = "1";
  document.getElementById("date").value = "";
  document.getElementById("timeStart").value = "";
  document.getElementById("timeEnd").value = "";
  document.getElementById("examDate").value = "";
}

    function addRow(sub) {
      const table = document.getElementById("scheduleTable");
      const row = table.insertRow();
      row.insertCell(0).innerText = sub.code;
      row.insertCell(1).innerText = sub.subject;
      row.insertCell(2).innerText = sub.credit;
      row.insertCell(3).innerText = sub.year;
      row.insertCell(4).innerText = sub.term;
      row.insertCell(5).innerText = formatDate(sub.date);
      row.insertCell(6).innerText = sub.timeStart;
      row.insertCell(7).innerText = sub.timeEnd;
      row.insertCell(8).innerText = formatDate(sub.examDate);

      const del = document.createElement("button");
      del.innerText = "ลบ";
      del.onclick = () => {
        subjects = subjects.filter(s => s !== sub);
        localStorage.setItem("schedule", JSON.stringify(subjects));
        row.remove();
        updateSummary();
      };
      row.insertCell(9).appendChild(del);
    }

    function updateSummary() {
      let total = 0;
      subjects.forEach(s => {
        total += s.credit;
      });
      document.getElementById("totalCredits").innerText = total.toFixed(2);
    }

   function downloadPDF() {
  const element = document.getElementById("capture");
  const opt = {
    margin:       0.5,
    filename:     'ตารางเรียน.pdf',
    image:        { type: 'jpeg', quality: 0.98 },
    html2canvas:  { scale: 2 },
    jsPDF:        { unit: 'in', format: 'a4', orientation: 'landscape' }
  };
  html2pdf().set(opt).from(element).save();
}

    function toggleCalendar() {
      const container = document.querySelector(".container");
      const calendarDiv = document.getElementById("calendarView");
      const backButton = document.getElementById("backButton");

      if (calendarDiv.style.display === "none") {
        container.style.display = "none";
        calendarDiv.style.display = "block";
        backButton.style.display = "block";
        showCalendar();
      } else {
        container.style.display = "block";
        calendarDiv.style.display = "none";
        backButton.style.display = "none";
      }
    }

    function showCalendar() {
      if (calendar) calendar.destroy();
      calendar = new FullCalendar.Calendar(document.getElementById('calendarView'), {
        initialView: 'dayGridMonth',
        locale: 'th',
        headerToolbar: {
          left: 'prev,next today',
          center: 'title',
          right: 'dayGridMonth,timeGridWeek'
        },
        events: subjects.map(s => ({
          title: `${s.subject} (${s.code})`,
          start: s.date + "T" + s.timeStart,
          end: s.date + "T" + s.timeEnd,
          allDay: false
        }))
      });
      calendar.render();
    }

    function formatDate(input) {
      const date = new Date(input);
      const days = ['อาทิตย์','จันทร์','อังคาร','พุธ','พฤหัสบดี','ศุกร์','เสาร์'];
      const months = ['มกราคม','กุมภาพันธ์','มีนาคม','เมษายน','พฤษภาคม','มิถุนายน',
        'กรกฎาคม','สิงหาคม','กันยายน','ตุลาคม','พฤศจิกายน','ธันวาคม'];
      return `${days[date.getDay()]} - ${months[date.getMonth()]} - ${date.getFullYear()}`;
    }

   function generateStudentView() {
  const body = document.getElementById('studentViewBody');
  body.innerHTML = '';

  subjects.forEach(sub => {
    const row = document.createElement('tr');
    row.innerHTML = `
      <td>${formatDate(sub.date).split(' - ')[0]}</td>
      <td>${sub.timeStart} - ${sub.timeEnd}</td>
      <td>${sub.code} (${sub.subject})</td>
      <td>${formatDate(sub.examDate)}</td>
    `;
    body.appendChild(row);
  });
}

    function showStudentView() {
      document.querySelector(".container").style.display = "none";
      document.getElementById("calendarView").style.display = "none";
      document.getElementById("backButton").style.display = "none";
      document.getElementById("studentTableView").style.display = "block";
      generateStudentView();
    }

    function toggleStudentView() {
      document.getElementById("studentTableView").style.display = "none";
      document.querySelector(".container").style.display = "block";
    }
    
    function downloadStudentPDF() {
  const element = document.getElementById("studentTableView");
  const opt = {
    margin:       0.5,
    filename:     'ตารางนักเรียน.pdf',
    image:        { type: 'jpeg', quality: 0.98 },
    html2canvas:  { scale: 2 },
    jsPDF:        { unit: 'in', format: 'a4', orientation: 'landscape' },
    pagebreak:    { mode: ['avoid-all', 'css', 'legacy'] }
  };
  html2pdf().set(opt).from(element).save();
}

  </script>
</body>
</html>
