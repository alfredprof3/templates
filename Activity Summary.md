
```dataviewjs
// 1. Get all pages created in the last 90 days
let pages = dv.pages()
    .filter(p => p.file.ctime >= moment().subtract(90, 'days'));

// 2. Gather data and group by Class -> Student -> Activity Title
let masterData = {}; 
let activitiesByClass = {}; 

for (let page of pages) {
    // Look specifically for tasks containing the activity key
    let tasks = page.file.tasks.filter(t => t.student && t.class && t.activity);
    
    if (tasks.length > 0) {
        for (let task of tasks) {
            let className = task.class;
            let studentName = task.student;
            let activityName = task.activity;
            
            if (!masterData[className]) masterData[className] = {};
            if (!masterData[className][studentName]) masterData[className][studentName] = {};
            if (!activitiesByClass[className]) activitiesByClass[className] = new Set();
            
            activitiesByClass[className].add(activityName);
            
            // Mark completion state
            masterData[className][studentName][activityName] = task.completed ? "✅ Done" : "❌ Missing";
        }
    }
}

// 3. Render a separate dashboard for each class period found
let classesFound = Object.keys(masterData).sort();

if (classesFound.length === 0) {
    dv.paragraph("*No student activities recorded in the last 7 days.*");
} else {
    for (let className of classesFound) {
        dv.header(2, `${className}`);
        
        let sortedActivities = Array.from(activitiesByClass[className]).sort();
        let totalActivities = sortedActivities.length;
        
        // Show total unique homework tasks reviewed this week
        dv.paragraph(`📚 **Total Activities Tracked:** ${totalActivities}`);
        
        let tableRows = [];
        let students = Object.keys(masterData[className]).sort();
        
        for (let student of students) {
            let row = [student];
            let doneCount = 0;
            
            for (let activity of sortedActivities) {
                let status = masterData[className][student][activity];
                row.push(status || "—");
                
                if (status === "✅ Done") {
                    doneCount++;
                }
            }
            
            // Calculate completion rate percentage
            let percentage = totalActivities > 0 ? Math.round((doneCount / totalActivities) * 100) : 0;
            
            // Highlight red if submission rate drops below 70%
            if (percentage < 70) {
                row.push(`<span style="color: #ff5555; font-weight: bold;">⚠️ ${percentage}% (${doneCount}/${totalActivities})</span>`);
            } else {
                row.push(`**${percentage}%** (${doneCount}/${totalActivities})`);
            }
            
            tableRows.push(row);
        }
        
        // Render final grid table
        dv.table(["Student Name", ...sortedActivities, "Submission Rate"], tableRows);
    }
}
```
