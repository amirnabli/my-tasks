# my-tasks
تطبيق مهامي اليومية هو أداة سهلة وبسيطة تساعدك على تنظيم مهامك وأعمالك اليومية مع تسجيل كل مهمة بالتاريخ. يمكنك إضافة المهام، تعديلها، حذفها، والبحث عنها بسهولة. يدعم التطبيق الوضع الليلي لتوفير راحة في القراءة، كما يمكنك طباعة قائمة مهامك كاملة. التطبيق متوفر باللغتين العربية والإنجليزية لتناسب جميع المستخدمين.<!DOCTYPE html>
<html lang="ar">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>تطبيق المهام مع دعم لغات وطباعة</title>
  <style>
    body {
      font-family: 'Tahoma', sans-serif;
      background-color: #f5f5f5;
      margin: 0; padding: 20px;
      direction: rtl;
      transition: 0.3s;
    }
    .dark-mode {
      background-color: #1e1e1e;
      color: #ddd;
    }
    h1 {
      text-align: center;
      color: teal;
    }
    .container {
      max-width: 600px;
      margin: auto;
      background: white;
      padding: 20px;
      border-radius: 12px;
      box-shadow: 0 4px 8px rgba(0,0,0,0.1);
    }
    input, button, select {
      padding: 10px;
      margin: 5px 0;
      font-size: 16px;
      border-radius: 6px;
      border: 1px solid #ccc;
      width: 100%;
      box-sizing: border-box;
    }
    button {
      background-color: teal;
      color: white;
      cursor: pointer;
      border: none;
      transition: 0.3s;
    }
    button:hover {
      background-color: #004d40;
    }
    .task {
      background: #e0f7fa;
      padding: 10px;
      margin-top: 10px;
      border-right: 4px solid teal;
      border-radius: 6px;
      position: relative;
    }
    .completed {
      text-decoration: line-through;
      opacity: 0.6;
    }
    .task-date {
      font-size: 13px;
      color: #555;
    }
    .btns {
      display: flex;
      gap: 10px;
      margin-top: 5px;
    }
    .btns button {
      flex: 1;
      padding: 6px;
      font-size: 14px;
    }
    .dark-mode .task {
      background: #333;
      border-color: #0ff;
    }
    .dark-mode .container {
      background: #2c2c2c;
    }
    .dark-mode input, .dark-mode button, .dark-mode select {
      background: #444;
      color: white;
      border-color: #666;
    }
    .dark-mode .task-date {
      color: #bbb;
    }
  </style>
</head>
<body>

  <h1 id="title">📋 مهامي اليومية</h1>

  <div class="container">
    <select id="langSelect" aria-label="اختيار اللغة">
      <option value="ar">العربية</option>
      <option value="en">English</option>
    </select>

    <input type="text" id="taskInput" placeholder="✏️ اكتب المهمة هنا" />
    <input type="date" id="dateInput" />
    <input type="text" id="searchInput" placeholder="🔍 ابحث عن مهمة..." />
    
    <button id="addBtn">➕ أضف المهمة</button>
    <button id="printBtn">🖨️ طباعة المهام</button>
    <button id="darkModeBtn">🌙 تبديل الوضع الليلي</button>

    <div id="taskList"></div>
  </div>

  <script>
    const texts = {
      ar: {
        title: "📋 مهامي اليومية",
        taskPlaceholder: "✏️ اكتب المهمة هنا",
        searchPlaceholder: "🔍 ابحث عن مهمة...",
        addBtn: "➕ أضف المهمة",
        printBtn: "🖨️ طباعة المهام",
        darkModeBtn: "🌙 تبديل الوضع الليلي",
        alertFill: "يرجى ملء المهمة والتاريخ.",
        completedBtn: ["تم", "إلغاء التحديد"],
        deleteBtn: "حذف",
        confirmDelete: "هل أنت متأكد من حذف المهمة؟",
      },
      en: {
        title: "📋 My Tasks",
        taskPlaceholder: "✏️ Write your task",
        searchPlaceholder: "🔍 Search tasks...",
        addBtn: "➕ Add Task",
        printBtn: "🖨️ Print Tasks",
        darkModeBtn: "🌙 Toggle Dark Mode",
        alertFill: "Please fill task and date.",
        completedBtn: ["Done", "Undo"],
        deleteBtn: "Delete",
        confirmDelete: "Are you sure to delete this task?",
      }
    };

    const langSelect = document.getElementById("langSelect");
    const taskInput = document.getElementById("taskInput");
    const dateInput = document.getElementById("dateInput");
    const searchInput = document.getElementById("searchInput");
    const addBtn = document.getElementById("addBtn");
    const printBtn = document.getElementById("printBtn");
    const darkModeBtn = document.getElementById("darkModeBtn");
    const taskList = document.getElementById("taskList");
    const title = document.getElementById("title");
    const body = document.body;

    let tasks = JSON.parse(localStorage.getItem("tasks")) || [];
    let currentLang = localStorage.getItem("lang") || "ar";

    // تعيين النصوص حسب اللغة
    function setLanguageTexts(lang) {
      title.textContent = texts[lang].title;
      taskInput.placeholder = texts[lang].taskPlaceholder;
      searchInput.placeholder = texts[lang].searchPlaceholder;
      addBtn.textContent = texts[lang].addBtn;
      printBtn.textContent = texts[lang].printBtn;
      darkModeBtn.textContent = texts[lang].darkModeBtn;
      langSelect.value = lang;
    }

    function saveTasks() {
      localStorage.setItem("tasks", JSON.stringify(tasks));
    }

    function displayTasks() {
      const search = searchInput.value.toLowerCase();
      taskList.innerHTML = "";

      tasks.forEach((task, index) => {
        if (!task.text.toLowerCase().includes(search)) return;

        const div = document.createElement("div");
        div.className = "task" + (task.completed ? " completed" : "");

        div.innerHTML = `
          <strong>${task.text}</strong>
          <div class="task-date">📅 ${task.date}</div>
          <div class="btns">
            <button onclick="toggleComplete(${index})">${task.completed ? texts[currentLang].completedBtn[1] : texts[currentLang].completedBtn[0]}</button>
            <button onclick="deleteTask(${index})">${texts[currentLang].deleteBtn}</button>
          </div>
        `;

        taskList.appendChild(div);
      });
    }

    function addTask() {
      const text = taskInput.value.trim();
      const date = dateInput.value;

      if (text === "" || date === "") {
        alert(texts[currentLang].alertFill);
        return;
      }

      tasks.push({ text, date, completed: false });
      tasks.sort((a, b) => new Date(a.date) - new Date(b.date));
      saveTasks();
      displayTasks();

      taskInput.value = "";
      dateInput.value = "";
    }

    function deleteTask(index) {
      if (confirm(texts[currentLang].confirmDelete)) {
        tasks.splice(index, 1);
        saveTasks();
        displayTasks();
      }
    }

    function toggleComplete(index) {
      tasks[index].completed = !tasks[index].completed;
      saveTasks();
      displayTasks();
    }

    function toggleDarkMode() {
      body.classList.toggle("dark-mode");
    }

    function printTasks() {
      let printContent = `<h1>${texts[currentLang].title}</h1><ul>`;
      tasks.forEach(task => {
        printContent += `<li>${task.text} - ${task.date} ${task.completed ? '(✓)' : ''}</li>`;
      });
      printContent += "</ul>";
      const printWindow = window.open('', '', 'width=600,height=400');
      printWindow.document.write(`<html><head><title>${texts[currentLang].title}</title></head><body>${printContent}</body></html>`);
      printWindow.document.close();
      printWindow.focus();
      printWindow.print();
      printWindow.close();
    }

    // حدث تغيير اللغة
    langSelect.addEventListener("change", () => {
      currentLang = langSelect.value;
      localStorage.setItem("lang", currentLang);
      setLanguageTexts(currentLang);
      displayTasks();
    });

    // أزرار وأحداث
    addBtn.addEventListener("click", addTask);
    printBtn.addEventListener("click", printTasks);
    darkModeBtn.addEventListener("click", toggleDarkMode);
    searchInput.addEventListener("input", displayTasks);

    // تهيئة
    setLanguageTexts(currentLang);
    displayTasks();
  </script>

</body>
</html>

