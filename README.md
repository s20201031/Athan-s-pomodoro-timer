# Athan-s-pomodoro-timer
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pomodoro Technique with Calendar</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 0;
            padding: 20px;
            background-color: #f0f8ff; /* Background color */
            color: #333;
        }
        h1 {
            text-align: center;
            color: #333; /* Title color */
        }
        #clock {
            text-align: center;
            font-size: 2em;
            margin-bottom: 20px;
            color: #333; /* Clock color */
        }
        .session-count {
            text-align: center;
            font-size: 1.2em;
            color: #333; /* Session count color */
        }
        .settings {
            display: flex;
            justify-content: space-between;
            margin-top: 20px;
        }
        .timer {
            margin: 10px 0;
            display: flex;
            justify-content: space-between;
            background-color: rgba(255, 255, 255, 0.8);
            padding: 10px;
            border-radius: 8px;
        }
        .todo, .habits {
            margin: 20px 0;
            background-color: rgba(255, 255, 255, 0.8);
            padding: 10px;
            border-radius: 8px;
        }
        .todo-zone {
            display: flex;
            flex-direction: column; /* Stack rows vertically */
            gap: 10px; /* Space between rows */
        }
        .todo-row {
            display: flex;
            justify-content: space-between;
        }
        textarea {
            width: 30%;
            height: 100px;
        }
        .timer-display {
            font-size: 1.5em;
        }
        .calendar {
            margin-top: 20px;
        }
        .calendar-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        .calendar-grid {
            display: grid;
            grid-template-columns: repeat(7, 1fr);
            gap: 5px;
            margin-top: 10px;
        }
        .calendar-day {
            background-color: #fff;
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 5px;
            position: relative;
        }
        .calendar-day.completed {
            background-color: #c8e6c9; /* Light green for completed tasks */
        }
        .task-list {
            margin-top: 5px;
            list-style-type: none;
            padding: 0;
        }
        .task-list li {
            cursor: pointer;
        }
        .edit-input {
            display: none;
            width: 100%;
            padding: 5px;
            margin-top: 5px;
        }
        .notification {
            text-align: center;
            font-size: 1.2em;
            color: #ff0000; /* Red color for notifications */
            margin-top: 10px;
        }
    </style>
</head>
<body>

    <h1>Pomodoro Technique Tracker</h1>
    <div id="clock"></div>
    <div class="session-count">Sessions Completed: <span id="sessionCount">0</span></div>
    <div class="notification" id="notification"></div> <!-- Notification area -->

    <div class="settings">
        <label>
            Pomodoro Duration (min):
            <input type="number" id="pomodoroDuration" value="25" min="1" />
        </label>
        <label>
            Break Duration (min):
            <input type="number" id="breakDuration" value="5" min="1" />
        </label>
    </div>

    <div class="timers">
        <div class="timer">
            <input type="number" class="timerInput" placeholder="Enter time in minutes" />
            <button class="startTimer">Start Timer</button>
            <button class="endTimer">End Timer</button>
            <button class="resetTimer">Reset Timer</button>
            <div class="timer-display">00:00</div>
        </div>
        <div class="timer">
            <input type="number" class="timerInput" placeholder="Enter time in minutes" />
            <button class="startTimer">Start Timer</button>
            <button class="endTimer">End Timer</button>
            <button class="resetTimer">Reset Timer</button>
            <div class="timer-display">00:00</div>
        </div>
        <div class="timer">
            <input type="number" class="timerInput" placeholder="Enter time in minutes" />
            <button class="startTimer">Start Timer</button>
            <button class="endTimer">End Timer</button>
            <button class="resetTimer">Reset Timer</button>
            <div class="timer-display">00:00</div>
        </div>
        <div class="timer">
            <input type="number" class="timerInput" placeholder="Enter time in minutes" />
            <button class="startTimer">Start Timer</button>
            <button class="endTimer">End Timer</button>
            <button class="resetTimer">Reset Timer</button>
            <div class="timer-display">00:00</div>
        </div>
    </div>

    <div class="todo">
        <h2>To-Do List</h2>
        <div class="todo-zone">
            <div class="todo-row">
                <textarea placeholder="緊急不重要" id="urgentNotImportant"></textarea>
            </div>
            <div class="todo-row">
                <textarea placeholder="緊急且重要" id="urgentImportant"></textarea>
            </div>
            <div class="todo-row">
                <textarea placeholder="重要不緊急" id="importantNotUrgent"></textarea>
            </div>
        </div>
        <button id="addTasks">Add Tasks to Calendar</button>
    </div>

    <div class="habits">
        <h2>Habits to Achieve</h2>
        <textarea placeholder="Enter habits here..." id="habitsText"></textarea>
    </div>

    <div class="calendar">
        <div class="calendar-header">
            <button id="prevMonth">Previous</button>
            <h2 id="currentMonth">September 2024</h2>
            <button id="nextMonth">Next</button>
        </div>
        <div class="calendar-grid" id="calendarGrid"></div>
    </div>

    <audio id="timerEndSound" src="https://www.soundjay.com/button/beep-07.wav"></audio> <!-- Sound alert -->

    <script>
        let currentDate = new Date();
        let sessionCount = 0;
        document.getElementById('sessionCount').innerText = sessionCount;

        // Function to update clock
        function updateClock() {
            const now = new Date();
            const options = { timeZone: 'Asia/Hong_Kong', hour: '2-digit', minute: '2-digit', second: '2-digit' };
            const timeString = now.toLocaleTimeString('en-US', options);
            document.getElementById('clock').innerText = timeString;
        }

        // Function to render the calendar
        function renderCalendar() {
            const month = currentDate.getMonth();
            const year = currentDate.getFullYear();
            const firstDay = new Date(year, month, 1);
            const lastDay = new Date(year, month + 1, 0);
            const daysInMonth = lastDay.getDate();
            const calendarGrid = document.getElementById('calendarGrid');
            const currentMonthDisplay = document.getElementById('currentMonth');

            // Clear the current grid
            calendarGrid.innerHTML = '';
            currentMonthDisplay.innerText = `${firstDay.toLocaleString('default', { month: 'long' })} ${year}`;

            // Fill the days in the grid
            for (let i = 1; i < firstDay.getDay(); i++) {
                calendarGrid.innerHTML += '<div class="calendar-day"></div>'; // Empty cells for days before the first
            }
            for (let day = 1; day <= daysInMonth; day++) {
                calendarGrid.innerHTML += `<div class="calendar-day" data-date="${year}-${month + 1}-${day}">
                    ${day}
                    <ul class="task-list" id="tasks-${year}-${month + 1}-${day}"></ul>
                </div>`;
            }
        }

        // Function to add tasks to the calendar
        document.getElementById('addTasks').addEventListener('click', () => {
            const urgentNotImportant = document.getElementById('urgentNotImportant').value;
            const urgentImportant = document.getElementById('urgentImportant').value;
            const importantNotUrgent = document.getElementById('importantNotUrgent').value;
            const tasks = [urgentNotImportant, urgentImportant, importantNotUrgent].filter(Boolean);
            const date = `${currentDate.getFullYear()}-${currentDate.getMonth() + 1}-${currentDate.getDate()}`;
            const taskList = document.getElementById(`tasks-${date}`);

            // Clear existing tasks
            taskList.innerHTML = '';

            tasks.forEach(task => {
                const li = document.createElement('li');
                li.textContent = task;
                li.onclick = () => editTask(li);
                taskList.appendChild(li);
            });
        });

        // Function to edit a task
        function editTask(taskElement) {
            const originalText = taskElement.textContent;
            const input = document.createElement('input');
            input.type = 'text';
            input.value = originalText;
            input.classList.add('edit-input');

            // Replace task text with input
            taskElement.innerHTML = '';
            taskElement.appendChild(input);
            input.focus();

            // Save changes on blur or enter key
            input.addEventListener('blur', () => saveTask(taskElement, input.value));
            input.addEventListener('keypress', (event) => {
                if (event.key === 'Enter') {
                    saveTask(taskElement, input.value);
                }
            });
        }

        // Function to save the edited task
        function saveTask(taskElement, newValue) {
            if (newValue.trim() === '') {
                taskElement.remove(); // Remove task if empty
            } else {
                taskElement.textContent = newValue;
                taskElement.onclick = () => editTask(taskElement); // Reassign click event
            }
        }

        // Navigation buttons for the calendar
        document.getElementById('prevMonth').addEventListener('click', () => {
            currentDate.setMonth(currentDate.getMonth() - 1);
            renderCalendar();
        });

        document.getElementById('nextMonth').addEventListener('click', () => {
            currentDate.setMonth(currentDate.getMonth() + 1);
            renderCalendar();
        });

        // Timer functionality (with start, end, and reset)
        const timers = document.querySelectorAll('.timer');
        timers.forEach(timer => {
            const input = timer.querySelector('.timerInput');
            const startButton = timer.querySelector('.startTimer');
            const endButton = timer.querySelector('.endTimer');
            const resetButton = timer.querySelector('.resetTimer');
            const display = timer.querySelector('.timer-display');
            const notificationDisplay = document.getElementById('notification'); // Notification element
            let timerInterval;
            let remainingTime = 0;

            // Start Timer
            startButton.addEventListener('click', () => {
                const minutes = parseInt(input.value);
                if (isNaN(minutes) || minutes <= 0) {
                    alert('Please enter a valid number of minutes.');
                    return;
                }

                remainingTime = minutes * 60; // Convert to seconds
                clearInterval(timerInterval);
                updateDisplay();
                notificationDisplay.innerText = `Timer started for ${minutes} minutes.`; // Visual notification

                timerInterval = setInterval(() => {
                    if (remainingTime <= 0) {
                        clearInterval(timerInterval);
                        sessionCount++;
                        document.getElementById('sessionCount').innerText = sessionCount;
                        alert('Timer is up!');
                        document.getElementById('timerEndSound').play();
                        notificationDisplay.innerText = 'Time is up!'; // Visual notification
                        display.innerText = '00:00';
                    } else {
                        remainingTime--;
                        updateDisplay();
                    }
                }, 1000);
            });

            // End Timer
            endButton.addEventListener('click', () => {
                clearInterval(timerInterval);
                notificationDisplay.innerText = 'Timer paused.'; // Visual notification
            });

            // Reset Timer
            resetButton.addEventListener('click', () => {
                clearInterval(timerInterval);
                remainingTime = 0;
                display.innerText = '00:00';
                input.value = ''; // Clear input
                notificationDisplay.innerText = ''; // Clear notifications
            });

            // Function to update the display
            function updateDisplay() {
                const mins = Math.floor(remainingTime / 60);
                const secs = remainingTime % 60;
                display.innerText = `${mins}:${secs < 10 ? '0' : ''}${secs}`;
            }
        });

        // Initial calls
        setInterval(updateClock, 1000);
        updateClock(); // Initial call
        renderCalendar(); // Render the calendar
    </script>

</body>
</html>
