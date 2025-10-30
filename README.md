# QUAN-LI-HOC-SINH
<!DOCTYPE html>
<html lang="vi">
<head>
    <meta charset="UTF-8">
    <title>Quản lý Học Sinh & TKB</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div class="container">
        <h1>Hệ Thống Quản Lý TKB</h1>

        <div class="card">
            <h2>Thêm Học Sinh</h2>
            <form id="add-student-form">
                <input type="text" id="student-name" placeholder="Tên học sinh mới" required>
                <button type="submit">Thêm Học Sinh</button>
            </form>
        </div>

        <div class="card">
            <h2>Quản lý lịch học</h2>
            
            <label for="student-select">Chọn học sinh:</label>
            <select id="student-select">
                <option value="">-- Vui lòng chọn --</option>
            </select>

            <form id="schedule-form">
                <input type="text" id="subject" placeholder="Tên môn học" required>
                <input type="text" id="time" placeholder="Thời gian" required>
                <button type="submit">Thêm lịch</button>
            </form>

            <hr>
            <h3 id="current-student-name">Thời khóa biểu của...</h3>
            <ul id="schedule-list">
                </ul>
        </div>
    <script src="main.js"></script>
</body>
</html>
body {
    font-family: Arial, sans-serif;
    background-color: #f0f2f5;
    color: #333;
    padding: 20px;
}

.container {
    max-width: 800px;
    margin: auto;
}

.card {
    background: #fff;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 2px 5px rgba(0,0,0,0.1);
    margin-bottom: 20px;
}

h1, h2, h3 {
    color: #0056b3;
}

input[type="text"], select {
    width: calc(100% - 22px);
    padding: 10px;
    margin-bottom: 10px;
    border: 1px solid #ccc;
    border-radius: 4px;
}

button {
    background: #007bff;
    color: #fff;
    padding: 10px 15px;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    width: 100%;
}
button:hover { background: #0056b3; }

ul {
    list-style-type: none;
    padding: 0;
}
li {
    background: #eef;
    padding: 10px;
    margin-bottom: 5px;
    border-radius: 4px;
    display: flex;
    justify-content: space-between;
}
li .delete-btn {
    color: red;
    cursor: pointer;
    font-weight: bold;
}
document.addEventListener('DOMContentLoaded', function() {
    
    // ========================================
    // --- PHẦN 1: GIẢ LẬP DATABASE ---
    // ========================================
    // Trong một ứng dụng thật, 'database' này sẽ nằm trên máy chủ (backend)
    // Dữ liệu sẽ được lưu vĩnh viễn.
    let database = {
        // Dữ liệu mẫu
        "hs_1": {
            name: "Nguyễn Văn A",
            schedule: [
                { subject: "Vật Lý", time: "Thứ 2 (7:00)" }
            ]
        },
        "hs_2": {
            name: "Trần Thị B",
            schedule: [
                { subject: "Hóa Học", time: "Thứ 3 (9:00)" }
            ]
        }
    };
    let currentStudentId = null; // Lưu ID của học sinh đang được chọn

    // Lấy các phần tử HTML
    const studentSelect = document.getElementById('student-select');
    const scheduleList = document.getElementById('schedule-list');
    const scheduleForm = document.getElementById('schedule-form');
    const studentForm = document.getElementById('add-student-form');
    const studentNameInput = document.getElementById('student-name');
    const currentStudentNameEl = document.getElementById('current-student-name');

    // ========================================
    // --- PHẦN 2: CÁC HÀM XỬ LÝ ---
    // ========================================

    // Hàm 1: Hiển thị (render) danh sách học sinh ra dropdown
    function renderStudentList() {
        studentSelect.innerHTML = '<option value="">-- Vui lòng chọn --</option>'; // Xóa danh sách cũ
        
        // Object.keys() lấy tất cả ID (hs_1, hs_2) từ database
        for (const studentId of Object.keys(database)) {
            const student = database[studentId];
            const option = document.createElement('option');
            option.value = studentId;
            option.textContent = student.name;
            studentSelect.appendChild(option);
        }
    }

    // Hàm 2: Hiển thị TKB của học sinh được chọn
    function renderSchedule(studentId) {
        scheduleList.innerHTML = ''; // Xóa TKB cũ

        if (!studentId || !database[studentId]) {
            currentStudentNameEl.textContent = 'Vui lòng chọn học sinh';
            return;
        }

        currentStudentId = studentId; // Cập nhật học sinh đang xem
        const student = database[studentId];
        currentStudentNameEl.textContent = `Thời khóa biểu của: ${student.name}`;

        if (student.schedule.length === 0) {
            scheduleList.innerHTML = '<li>Chưa có lịch học.</li>';
            return;
        }

        student.schedule.forEach((item, index) => {
            const li = document.createElement('li');
            li.innerHTML = `
                <strong>${item.subject}</strong> - <span>${item.time}</span>
                <span class="delete-btn" data-index="${index}">X</span>
            `;
            scheduleList.appendChild(li);
        });
    }

    // ========================================
    // --- PHẦN 3: LẮNG NGHE SỰ KIỆN ---
    // ========================================

    // Sự kiện 1: Khi thêm học sinh mới
    studentForm.addEventListener('submit', function(e) {
        e.preventDefault();
        const newName = studentNameInput.value;
        
        // Tạo một ID ngẫu nhiên đơn giản
        const newId = 'hs_' + Date.now(); 
        
        // Thêm vào "database"
        database[newId] = {
            name: newName,
            schedule: [] // Lịch học ban đầu rỗng
        };

        studentNameInput.value = '';
        renderStudentList(); // Cập nhật lại danh sách dropdown
        alert(`Đã thêm học sinh ${newName}!`);
    });

    // Sự kiện 2: Khi người dùng thay đổi lựa chọn học sinh
    studentSelect.addEventListener('change', function() {
        const selectedId = studentSelect.value;
        renderSchedule(selectedId);
    });

    // Sự kiện 3: Khi thêm lịch học mới (cho học sinh đang chọn)
    scheduleForm.addEventListener('submit', function(e) {
        e.preventDefault();
        
        if (!currentStudentId) {
            alert("Bạn phải chọn một học sinh trước!");
            return;
        }

        const subject = document.getElementById('subject').value;
        const time = document.getElementById('time').value;

        // Thêm lịch học mới vào "database"
        database[currentStudentId].schedule.push({ subject, time });

        // Hiển thị lại TKB
        renderSchedule(currentStudentId);

        // Xóa ô input
        document.getElementById('subject').value = '';
        document.getElementById('time').value = '';
    });

    // Sự kiện 4: Khi bấm nút X (Xóa TKB)
    scheduleList.addEventListener('click', function(e) {
        if (e.target.classList.contains('delete-btn')) {
            const itemIndex = e.target.getAttribute('data-index');
            
            // Xóa mục khỏi "database"
            database[currentStudentId].schedule.splice(itemIndex, 1);
            
            // Hiển thị lại TKB
            renderSchedule(currentStudentId);
        }
    });

    // --- Khởi chạy lần đầu ---
    renderStudentList();
    renderSchedule(null);
});
