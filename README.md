# Task-Trekker
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Schedule Planner</title>
    <style>
        body {
            font-family: Arial, sans-serif;
        }

        h1 {
            color: #3498db;
        }

        #taskList {
            list-style-type: none;
            padding: 0;
        }

        .task {
            background-color: #3498db;
            color: #fff;
            padding: 5px;
            margin: 5px;
            border-radius: 5px;
            cursor: pointer;
        }

        #schedule {
            border-collapse: collapse;
            width: 100%;
        }

        #schedule th, #schedule td {
            border: 1px solid #ddd;
            padding: 8px;
            text-align: left;
        }

        #schedule th {
            background-color: #3498db;
            color: #fff;
        }

        #overviewTable {
            border-collapse: collapse;
            width: 100%;
            margin-top: 20px;
        }

        #overviewTable th, #overviewTable td {
            border: 1px solid #ddd;
            padding: 8px;
            text-align: left;
        }

        #overviewTable th {
            background-color: #3498db;
            color: #fff;
        }
    </style>
</head>
<body>
    <h1>Schedule Planner</h1>

    <label for="deadline">Enter Overall Deadline:</label>
    <input type="text" id="deadline" placeholder="MM/DD/YYYY">
    
    <button onclick="addTask()">Add Task</button>

    <h2>Tasks:</h2>
    <ul id="taskList"></ul>

    <button onclick="generateSchedule()">Generate Schedule</button>

    <h2>Generated Schedule:</h2>
    <table id="schedule">
        <thead>
            <tr>
                <th>Task</th>
                <th>Type</th>
                <th>Deadline</th>
                <th>Suggested Start Date</th>
                <th>Suggested Time</th>
                <th>Action</th>
            </tr>
        </thead>
        <tbody id="scheduleBody"></tbody>
    </table>

    <h2>Weekly Overview:</h2>
    <table id="overviewTable">
        <thead>
            <tr>
                <th>Day</th>
                <th>Start Date</th>
                <th>Month</th>
                <th>Task</th>
                <th>Time Distribution</th>
            </tr>
        </thead>
        <tbody id="overviewBody"></tbody>
    </table>

    <script>
        let tasks = [];

        function addTask() {
            const taskName = prompt('Enter the task name:');
            if (!taskName) {
                return; // User canceled task entry
            }

            const taskType = prompt('Enter the task type (Assessment or Exam):');
            if (!taskType) {
                return; // User canceled task type entry
            }

            const taskDeadline = new Date(prompt('Enter the task deadline (MM/DD/YYYY):'));
            if (isNaN(taskDeadline.getTime())) {
                alert('Please enter a valid deadline in MM/DD/YYYY format.');
                return;
            }

            tasks.push({ name: taskName, type: taskType, deadline: taskDeadline });
            updateTaskList();
        }

        function updateTaskList() {
            const taskList = document.getElementById('taskList');
            taskList.innerHTML = '';

            tasks.forEach(task => {
                const listItem = document.createElement('li');
                listItem.textContent = `${task.name} (${task.type}) - ${task.deadline.toLocaleDateString()}`;
                listItem.onclick = () => removeTask(task);
                taskList.appendChild(listItem);
            });
        }

        function generateSchedule() {
            const overallDeadlineInput = document.getElementById('deadline');
            const overallDeadline = new Date(overallDeadlineInput.value);

            if (isNaN(overallDeadline.getTime())) {
                alert('Please enter a valid overall deadline in MM/DD/YYYY format.');
                return;
            }

            tasks.sort((a, b) => a.deadline - b.deadline);

            const scheduleBody = document.getElementById('scheduleBody');
            scheduleBody.innerHTML = '';

            const overviewBody = document.getElementById('overviewBody');
            overviewBody.innerHTML = '';

            let currentDate = new Date(overallDeadline);
            currentDate.setDate(currentDate.getDate() - 28); // Suggest start date four weeks before the deadline

            let weeksBeforeDeadline = 4;

            tasks.forEach(task => {
                const suggestedStartDate = new Date(task.deadline);
                suggestedStartDate.setDate(suggestedStartDate.getDate() - weeksBeforeDeadline * 7);

                const suggestedStartTime = new Date(suggestedStartDate);
                suggestedStartTime.setHours(16 + Math.floor(Math.random() * 5), Math.floor(Math.random() * 60)); // Suggested start time: 4:00 PM to 9:00 PM
                const suggestedEndTime = new Date(suggestedStartTime);
                suggestedEndTime.setHours(suggestedStartTime.getHours() + 1, 30); // Suggested end time: 5:30 PM

                if (suggestedStartDate < currentDate) {
                    suggestedStartDate.setDate(currentDate.getDate());
                }

                if (task.deadline > overallDeadline) {
                    task.deadline = overallDeadline;
                }

                const newRow = scheduleBody.insertRow();
                newRow.insertCell(0).textContent = task.name;
                newRow.insertCell(1).textContent = task.type;
                newRow.insertCell(2).textContent = task.deadline.toLocaleDateString();
                newRow.insertCell(3).textContent = suggestedStartDate.toLocaleString();
                newRow.insertCell(4).textContent = `${suggestedStartTime.toLocaleTimeString()} - ${suggestedEndTime.toLocaleTimeString()}`;
                newRow.insertCell(5).innerHTML = '<button onclick="finishTask(\'' + task.name + '\')">Finish Task</button>';

                currentDate = new Date(suggestedEndTime);

                // Update weekly overview
                const dayOfWeek = suggestedStartDate.toLocaleDateString('en-US', { weekday: 'long' });
                const month = suggestedStartDate.toLocaleDateString('en-US', { month: 'long' });

                const overviewRow = overviewBody.insertRow();
                overviewRow.insertCell(0).textContent = dayOfWeek;
                overviewRow.insertCell(1).textContent = suggestedStartDate.toLocaleDateString();
                overviewRow.insertCell(2).textContent = month;
                overviewRow.insertCell(3).textContent = task.name;
                overviewRow.insertCell(4).textContent = `Distribute ${1.5} hours over the next few days`;

                // Distribute task over the next few days until the deadline
                while (currentDate < task.deadline) {
                    const nextDay = new Date(currentDate);
                    nextDay.setDate(nextDay.getDate() + 1);

                    const distributeRow = overviewBody.insertRow();
                    distributeRow.insertCell(0).textContent = nextDay.toLocaleDateString('en-US', { weekday: 'long' });
                    distributeRow.insertCell(1).textContent = nextDay.toLocaleDateString();
                    distributeRow.insertCell(2).textContent = nextDay.toLocaleDateString('en-US', { month: 'long' });
                    distributeRow.insertCell(3).textContent = task.name;
                    distributeRow.insertCell(4).textContent = `Distribute ${1.5} hours`;

                    currentDate = new Date(nextDay);
                }
            });
        }

        function removeTask(task) {
            const index = tasks.findIndex(t => t.name === task.name);
            if (index !== -1) {
                tasks.splice(index, 1);
                updateTaskList();
            }
        }

        function finishTask(taskName) {
            const task = tasks.find(t => t.name === taskName);
            if (task) {
                removeTask(task);
                generateSchedule();
            }
        }
    </script>
</body>
</html>
