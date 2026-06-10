<%*
const storagePath = "Student-Rosters.json";
if (!(await tp.file.exists(storagePath))) {
    new Notice("Database not found!"); return;
}
let content = await app.vault.adapter.read(storagePath);
let db = JSON.parse(content);
let classes = Object.keys(db.classes || {}).sort();

// 1. Select the class for report generation
let chosenClass = await tp.system.suggester(classes, classes);
if (!chosenClass) return;

// Safely access the Dataview API inside Templater
const dR = app.plugins.plugins.dataview?.api;
if (!dR) {
    new Notice("Dataview plugin is not activated!"); return;
}

// 2. Fetch all pages in your vault safely using the verified API instance
let pages = dR.pages();
let attendanceData = {};
let activityData = {};
let totalSessions = 0;
let totalActivities = new Set();

// 3. Scan historical files to calculate semester metrics
for (let page of pages) {
    let tasks = page.file.tasks.filter(t => t.student && t.class === chosenClass);
    if (tasks.length > 0) {
        let isAttendancePage = tasks.some(t => !t.activity);
        if (isAttendancePage) totalSessions++;

        for (let task of tasks) {
            let student = task.student;
            if (task.activity) {
                // Track assignments
                totalActivities.add(task.activity);
                if (!activityData[student]) activityData[student] = 0;
                if (task.completed) activityData[student]++;
            } else {
                // Track attendance
                if (!attendanceData[student]) attendanceData[student] = 0;
                if (task.completed) attendanceData[student]++;
            }
        }
    }
}

let totalActivitiesCount = totalActivities.size;
let studentIds = db.classes[chosenClass].studentIds;
let studentNames = studentIds.map(id => db.students[id].name).sort();

// 4. Print individual Report Cards
tR += `# 🎓 Academic Report Cards: ${chosenClass}\n`;
tR += `🏫 **Institution:** ${db.classes[chosenClass].university} | 🗓️ **Term Ending:** ${tp.date.now("MMMM YYYY")}\n\n---\n\n`;

// Define placeholder labels to intercept matching placeholder records
const bypassLabels = ["na", "n/a", "none", "no email", "not applicable"];

for (let name of studentNames) {
    // Find student details from our relational global records
    let studentId = Object.keys(db.students).find(id => db.students[id].name === name);
    let rawEmail = db.students[studentId]?.email ? db.students[studentId].email.trim() : "N/A";
    let emailDisplay = "";

    // FIXED: Evaluate if email matches an NA token block string
    if (bypassLabels.includes(rawEmail.toLowerCase())) {
        emailDisplay = `*${rawEmail.toUpperCase()}*`;
    } else {
        emailDisplay = rawEmail;
    }
    
    // Calculations
    let attended = attendanceData[name] || 0;
    let attendanceRate = totalSessions > 0 ? Math.round((attended / totalSessions) * 100) : 0;
    
    let completedActs = activityData[name] || 0;
    let activityRate = totalActivitiesCount > 0 ? Math.round((completedActs / totalActivitiesCount) * 100) : 0;
    
    // Status badges based on your 70% threshold
    let attendanceBadge = attendanceRate >= 70 ? "✅" : "⚠️ CRITICAL (Below 70%)";
    let activityBadge = activityRate >= 70 ? "✅" : "❌ INCOMPLETE (Below 70%)";

    // Markdown Report Card Format
    tR += `## 👤 ${name}\n`;
    tR += `* **Enrollment ID:** \`${studentId}\`\n`;
    tR += `* **Email:** ${emailDisplay}\n\n`; // FIXED: Outputs formatted placeholder or raw email address
    tR += `### 📈 Summary\n`;
    tR += `| Metric | Progress | Percentage | Status |\n`;
    tR += `| --- | --- | --- | --- |\n`;
    tR += `| **Class Attendance** | ${attended} / ${totalSessions} sessions | **${attendanceRate}%** | ${attendanceBadge} |\n`;
    tR += `| **Activities Turned In** | ${completedActs} / ${totalActivitiesCount} tasks | **${activityRate}%** | ${activityBadge} |\n\n`;
    tR += `**Instructor Comments:**\n> \n\n`;
    tR += `---\n\n`;
}
new Notice(`Generated report cards for ${chosenClass}!`);
-%>
