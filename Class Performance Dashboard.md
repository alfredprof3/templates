
```dataviewjs
// 1. Configuration: Path to save the rosters JSON file
const storagePath = "Attendance-Rosters.json";
const fileExists = app.vault.getAbstractFileByPath(storagePath);

if (!fileExists) {
    dv.paragraph("⚠️ *No database found yet. Run your Roster Manager template first!*");
} else {
    // 2. Load and parse the central database layout
    let content = await app.vault.adapter.read(storagePath);
    let db = JSON.parse(content);
    let classes = Object.keys(db.classes || {}).sort();

    if (classes.length === 0) {
        dv.paragraph("*No active classes found in database.*");
    } else {
        dv.header(2, "📊 Faculty Analytics: Semester Class Averages");
        
        // 3. Scan all historical logs in the vault to collect data points
        let pages = dv.pages();
        let classAttendance = {};
        let classActivities = {};

        for (let page of pages) {
            // Find tasks associated with our tracking system
            let tasks = page.file.tasks.filter(t => t.student && t.class);
            
            if (tasks.length > 0) {
                for (let task of tasks) {
                    let className = task.class;
                    
                    // Initialize metrics structure for the class if missing
                    if (!classAttendance[className]) {
                        classAttendance[className] = { totalCheckboxes: 0, checkedCount: 0 };
                    }
                    if (!classActivities[className]) {
                        classActivities[className] = { totalCheckboxes: 0, checkedCount: 0 };
                    }

                    // Sort data points into Attendance vs Activity bins
                    if (task.activity) {
                        classActivities[className].totalCheckboxes++;
                        if (task.completed) classActivities[className].checkedCount++;
                    } else {
                        classAttendance[className].totalCheckboxes++;
                        if (task.completed) classAttendance[className].checkedCount++;
                    }
                }
            }
        }

        // 4. Generate the dashboard comparison table
        let tableRows = [];

        for (let className of classes) {
            let classInfo = db.classes[className];
            let totalStudents = classInfo.studentIds ? classInfo.studentIds.length : 0;

            // Calculate Attendance Average
            let attData = classAttendance[className];
            let attAvg = "—";
            if (attData && attData.totalCheckboxes > 0) {
                let pct = Math.round((attData.checkedCount / attData.totalCheckboxes) * 100);
                // Highlight red if class attendance drops below your 70% threshold
                attAvg = pct < 70 ? `<span style="color: #ff5555; font-weight: bold;">⚠️ ${pct}%</span>` : `**${pct}%**`;
            }

            // Calculate Activity Submission Average
            let actData = classActivities[className];
            let actAvg = "—";
            if (actData && actData.totalCheckboxes > 0) {
                let pct = Math.round((actData.checkedCount / actData.totalCheckboxes) * 100);
                // Highlight red if homework submissions drop below 70%
                actAvg = pct < 70 ? `<span style="color: #ff5555; font-weight: bold;">⚠️ ${pct}%</span>` : `**${pct}%**`;
            }

            tableRows.push([
                className,
                classInfo.university,
                classInfo.semester,
                totalStudents,
                attAvg,
                actAvg
            ]);
        }

        // 5. Render the final metric table
        dv.table(
            ["Course Name", "Institution", "Semester Level", "Total Roster", "Avg Attendance", "Avg Submissions"], 
            tableRows
        );
    }
}
```
