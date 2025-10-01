# study-smarter
this is a website that helps the students to learn through online. Created by webtech developers under C. E.O Fredrick Mbugua who is a full stack developer
[index.html](https://github.com/user-attachments/files/22648028/index.html)
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Smart Study Planner — Frontend</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <style>
    /* small extra styles for calendar cells */
    .calendar-grid { grid-template-columns: repeat(7, 1fr); }
    .task-dot { width: 8px; height: 8px; border-radius: 999px; display: inline-block; }
  </style>
</head>
<body class="bg-gray-50 dark:bg-gray-900 text-gray-800 dark:text-gray-100 min-h-screen">
  <div class="max-w-6xl mx-auto p-4">
    <header class="flex items-center justify-between gap-4 mb-6">
      <div>
        <h1 class="text-2xl font-semibold">Smart Study Planner</h1>
        <p class="text-sm text-gray-600 dark:text-gray-300">Plan your day, track progress, and stay focused.</p>
      </div>

      <div class="flex items-center gap-3">
        <button id="themeToggle" class="px-3 py-1 rounded-md border dark:border-gray-700">Toggle Theme</button>
        <button id="exportBtn" class="px-3 py-1 rounded-md bg-blue-600 text-white">Export JSON</button>
      </div>
    </header>

    <main class="grid md:grid-cols-3 gap-6">
      <!-- Left: Add Task / List -->
      <section class="md:col-span-1 bg-white dark:bg-gray-800 rounded-2xl p-4 shadow">
        <h2 class="text-lg font-medium mb-3">Add Task</h2>
        <form id="taskForm" class="space-y-3">
          <input id="title" class="w-full rounded-md border px-3 py-2 bg-transparent" placeholder="Task title" required />
          <textarea id="notes" class="w-full rounded-md border px-3 py-2 bg-transparent" placeholder="Notes (optional)" rows="2"></textarea>

          <div class="flex gap-2">
            <input id="date" type="date" class="rounded-md border px-3 py-2 bg-transparent" />
            <select id="priority" class="rounded-md border px-3 py-2 bg-transparent">
              <option value="low">Low</option>
              <option value="medium" selected>Medium</option>
              <option value="high">High</option>
            </select>
          </div>

          <div class="flex gap-2">
            <button type="submit" class="flex-1 px-4 py-2 rounded-md bg-green-600 text-white">Add Task</button>
            <button id="clearBtn" type="button" class="px-4 py-2 rounded-md border">Clear</button>
          </div>
        </form>

        <hr class="my-4 border-gray-200 dark:border-gray-700" />

        <h3 class="text-md font-medium mb-2">Tasks</h3>
        <ul id="taskList" class="space-y-2 max-h-72 overflow-auto"></ul>
      </section>

      <!-- Middle: Calendar -->
      <section class="md:col-span-2 bg-white dark:bg-gray-800 rounded-2xl p-4 shadow">
        <div class="flex items-center justify-between mb-4">
          <div>
            <h2 id="monthYear" class="text-xl font-semibold"></h2>
            <p class="text-sm text-gray-500 dark:text-gray-300">Click a date to see tasks for that day.</p>
          </div>
          <div class="flex gap-2">
            <button id="prevMonth" class="px-3 py-1 rounded-md border">Prev</button>
            <button id="nextMonth" class="px-3 py-1 rounded-md border">Next</button>
          </div>
        </div>

        <div class="grid calendar-grid gap-2 text-sm mb-2">
          <div class="text-center font-medium">Sun</div>
          <div class="text-center font-medium">Mon</div>
          <div class="text-center font-medium">Tue</div>
          <div class="text-center font-medium">Wed</div>
          <div class="text-center font-medium">Thu</div>
          <div class="text-center font-medium">Fri</div>
          <div class="text-center font-medium">Sat</div>
        </div>

        <div id="calendar" class="grid calendar-grid gap-2"></div>

        <hr class="my-4 border-gray-200 dark:border-gray-700" />

        <div>
          <h3 class="text-lg font-medium mb-2">Tasks for <span id="selectedDateLabel">—</span></h3>
          <ul id="dayTasks" class="space-y-2 max-h-44 overflow-auto"></ul>
        </div>
      </section>
    </main>

    <footer class="mt-6 text-xs text-gray-500 dark:text-gray-400">&copy;COPYRIGHT 2025 || WEBTECH DEVELOPERS</footer>
  </div>

  <script>
    // ------------------ Storage helpers ------------------
    const STORAGE_KEY = 'smartPlanner.tasks.v1';
    function loadTasks() {
      try {
        return JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]');
      } catch (e) {
        console.error('Failed to parse tasks', e);
        return [];
      }
    }
    function saveTasks(tasks) { localStorage.setItem(STORAGE_KEY, JSON.stringify(tasks)); }

    // ------------------ State ------------------
    let tasks = loadTasks();
    let today = new Date();
    let viewYear = today.getFullYear();
    let viewMonth = today.getMonth(); // 0 indexed
    let selectedDate = null; // yyyy-mm-dd

    // ------------------ UI Elements ------------------
    const taskForm = document.getElementById('taskForm');
    const titleInput = document.getElementById('title');
    const notesInput = document.getElementById('notes');
    const dateInput = document.getElementById('date');
    const priorityInput = document.getElementById('priority');
    const taskList = document.getElementById('taskList');
    const calendarEl = document.getElementById('calendar');
    const monthYearLabel = document.getElementById('monthYear');
    const selectedDateLabel = document.getElementById('selectedDateLabel');
    const dayTasks = document.getElementById('dayTasks');
    const prevBtn = document.getElementById('prevMonth');
    const nextBtn = document.getElementById('nextMonth');
    const exportBtn = document.getElementById('exportBtn');
    const clearBtn = document.getElementById('clearBtn');
    const themeToggle = document.getElementById('themeToggle');

    // ------------------ Utils ------------------
    function formatDateString(d) {
      const yyyy = d.getFullYear();
      const mm = String(d.getMonth()+1).padStart(2,'0');
      const dd = String(d.getDate()).padStart(2,'0');
      return `${yyyy}-${mm}-${dd}`;
    }

    function tasksForDate(dateStr) {
      return tasks.filter(t => t.date === dateStr);
    }

    function uid() { return Date.now().toString(36) + Math.random().toString(36).slice(2,6); }

    // ------------------ Renderers ------------------
    function renderTaskList() {
      taskList.innerHTML = '';
      if (!tasks.length) {
        taskList.innerHTML = '<li class="text-sm text-gray-500">No tasks yet.</li>';
        return;
      }
      tasks.slice().reverse().forEach(t => {
        const li = document.createElement('li');
        li.className = 'p-2 rounded-md border flex items-start justify-between gap-2';
        li.innerHTML = `
          <div class="flex-1">
            <div class="flex items-center gap-2">
              <strong>${escapeHtml(t.title)}</strong>
              <span class="text-xs text-gray-500">${t.date || 'No date'}</span>
              <span class="ml-2 px-2 py-0.5 text-xs rounded ${t.priority==='high'? 'bg-red-100 text-red-700': t.priority==='medium' ? 'bg-yellow-100 text-yellow-700':'bg-green-100 text-green-700'}">${t.priority}</span>
            </div>
            <div class="text-sm text-gray-600 dark:text-gray-300">${escapeHtml(t.notes || '')}</div>
          </div>
          <div class="flex flex-col items-end gap-2">
            <button data-id="${t.id}" class="markBtn px-2 py-1 rounded-md border text-sm">${t.done? 'Undo':'Done'}</button>
            <button data-id="${t.id}" class="editBtn px-2 py-1 rounded-md border text-sm">Edit</button>
            <button data-id="${t.id}" class="delBtn px-2 py-1 rounded-md border text-sm">Delete</button>
          </div>
        `;
        if (t.done) li.classList.add('opacity-60','line-through');
        taskList.appendChild(li);
      });
    }

    function renderCalendar() {
      monthYearLabel.textContent = new Date(viewYear, viewMonth).toLocaleString(undefined,{month:'long', year:'numeric'});
      calendarEl.innerHTML = '';

      const firstDay = new Date(viewYear, viewMonth, 1).getDay();
      const daysInMonth = new Date(viewYear, viewMonth+1, 0).getDate();

      // fill blanks
      for (let i=0;i<firstDay;i++) {
        const cell = document.createElement('div');
        cell.className = 'p-3 rounded-md border border-transparent h-20';
        calendarEl.appendChild(cell);
      }

      for (let d=1; d<=daysInMonth; d++) {
        const dateStr = `${viewYear}-${String(viewMonth+1).padStart(2,'0')}-${String(d).padStart(2,'0')}`;
        const cell = document.createElement('button');
        cell.className = 'text-left p-3 rounded-md border h-20 flex flex-col justify-between';
        cell.innerHTML = `<div class="flex items-center justify-between"><span class="text-sm font-medium">${d}</span></div><div class="text-xs flex gap-1" aria-hidden></div>`;

        const dayTasksArr = tasksForDate(dateStr);
        const dots = cell.querySelector('div[aria-hidden]');
        dayTasksArr.slice(0,3).forEach(t=>{
          const dot = document.createElement('span');
          dot.className = 'task-dot';
          dot.title = t.title;
          dot.style.background = t.priority==='high' ? '#ef4444' : t.priority==='medium'? '#f59e0b' : '#10b981';
          dots.appendChild(dot);
        });

        if (dateStr === formatDateString(new Date())) {
          cell.classList.add('ring','ring-blue-200');
        }

        cell.addEventListener('click', ()=>{
          selectedDate = dateStr;
          selectedDateLabel.textContent = selectedDate;
          renderDayTasks();
        });

        calendarEl.appendChild(cell);
      }
    }

    function renderDayTasks() {
      dayTasks.innerHTML = '';
      if (!selectedDate) return;
      const arr = tasksForDate(selectedDate);
      if (!arr.length) { dayTasks.innerHTML = '<li class="text-sm text-gray-500">No tasks for this date.</li>'; return; }
      arr.forEach(t=>{
        const li = document.createElement('li');
        li.className = 'p-2 rounded-md border flex items-center justify-between gap-2';
        li.innerHTML = `
          <div>
            <div class="text-sm font-medium">${escapeHtml(t.title)}</div>
            <div class="text-xs text-gray-600 dark:text-gray-300">${escapeHtml(t.notes||'')}</div>
          </div>
          <div class="flex gap-2">
            <button data-id="${t.id}" class="markBtn px-2 py-1 rounded-md border text-sm">${t.done? 'Undo':'Done'}</button>
            <button data-id="${t.id}" class="delBtn px-2 py-1 rounded-md border text-sm">Delete</button>
          </div>
        `;
        dayTasks.appendChild(li);
      });
    }

    // ------------------ Events ------------------
    taskForm.addEventListener('submit', e=>{
      e.preventDefault();
      const title = titleInput.value.trim();
      if (!title) return;
      const newTask = {
        id: uid(),
        title,
        notes: notesInput.value.trim(),
        date: dateInput.value || null,
        priority: priorityInput.value || 'medium',
        done: false,
        createdAt: new Date().toISOString()
      };
      tasks.push(newTask);
      saveTasks(tasks);
      titleInput.value = '';
      notesInput.value = '';
      dateInput.value = '';
      renderTaskList();
      renderCalendar();
    });

    // delegation for task actions
    document.addEventListener('click', (e)=>{
      const markBtn = e.target.closest('.markBtn');
      const delBtn = e.target.closest('.delBtn');
      const editBtn = e.target.closest('.editBtn');
      if (markBtn) {
        const id = markBtn.dataset.id;
        toggleDone(id);
      } else if (delBtn) {
        const id = delBtn.dataset.id;
        deleteTask(id);
      } else if (editBtn) {
        const id = editBtn.dataset.id;
        startEdit(id);
      }
    });

    prevBtn.addEventListener('click', ()=>{ viewMonth--; if (viewMonth<0){ viewMonth=11; viewYear--; } renderCalendar(); });
    nextBtn.addEventListener('click', ()=>{ viewMonth++; if (viewMonth>11){ viewMonth=0; viewYear++; } renderCalendar(); });

    exportBtn.addEventListener('click', ()=>{
      const dataStr = JSON.stringify(tasks, null, 2);
      const blob = new Blob([dataStr], {type:'application/json'});
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      a.href = url;
      a.download = 'smart-planner-tasks.json';
      a.click();
      URL.revokeObjectURL(url);
    });

    clearBtn.addEventListener('click', ()=>{
      titleInput.value = '';
      notesInput.value = '';
      dateInput.value = '';
      priorityInput.value = 'medium';
    });

    themeToggle.addEventListener('click', ()=>{
      document.documentElement.classList.toggle('dark');
      // remember preference
      localStorage.setItem('smartPlanner.theme', document.documentElement.classList.contains('dark') ? 'dark' : 'light');
    });

    // ------------------ Task operations ------------------
    function toggleDone(id){
      tasks = tasks.map(t => t.id === id ? {...t, done: !t.done} : t);
      saveTasks(tasks); renderTaskList(); renderDayTasks(); renderCalendar();
    }
    function deleteTask(id){
      if (!confirm('Delete this task?')) return;
      tasks = tasks.filter(t=>t.id !== id);
      saveTasks(tasks); renderTaskList(); renderDayTasks(); renderCalendar();
    }
    function startEdit(id){
      const t = tasks.find(x=>x.id===id);
      if (!t) return;
      // populate form for editing (simple approach: delete original and let user re-add)
      if (!confirm('This will load the task into the form for editing and remove the original. Continue?')) return;
      titleInput.value = t.title; notesInput.value = t.notes; dateInput.value = t.date || '';
      priorityInput.value = t.priority || 'medium';
      tasks = tasks.filter(x=>x.id!==id);
      saveTasks(tasks);
      renderTaskList(); renderCalendar();
    }

    // ------------------ Safety helpers ------------------
    function escapeHtml(unsafe) {
      return String(unsafe)
        .replace(/&/g, '&amp;')
        .replace(/</g, '&lt;')
        .replace(/>/g, '&gt;')
        .replace(/"/g, '&quot;')
        .replace(/'/g, '&#039;');
    }

    // ------------------ Init ------------------
    function applySavedTheme(){
      const t = localStorage.getItem('smartPlanner.theme') || (window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light');
      if (t === 'dark') document.documentElement.classList.add('dark');
    }

    applySavedTheme();
    renderTaskList();
    renderCalendar();

    // When a date is clicked, default to today if none
    selectedDate = formatDateString(new Date());
    selectedDateLabel.textContent = selectedDate;
    renderDayTasks();
  </script>
</body>
</html>
