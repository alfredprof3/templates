
```dataviewjs
// 1. Get all pages created in the last 90 days
let pages = dv.pages()
    .filter(p => p.file.ctime >= moment().subtract(90, 'days'));

// 2. Gather attendance tasks and group them by Class -> Student -> Date
let masterData = {}; 
let datesByClass = {}; 

for (let page of pages) {
    let tasks = page.file.tasks.filter(t => t.student && t.class);
    
    if (tasks.length > 0) {
        // FIXED: Corrected the regex optional chaining syntax here
        let dateStr = page.file.name.match(/\d{4}-\d{2}-\d{2}/)?.[0] || page.file.ctime.toFormat("dd-MM-yyyy");
        
        for (let task of tasks) {
            let className = task.class;
            let studentName = task.student;
            
            if (!masterData[className]) masterData[className] = {};
            if (!masterData[className][studentName]) masterData[className][studentName] = {};
            if (!datesByClass[className]) datesByClass[className] = new Set();
            
            datesByClass[className].add(dateStr);
            masterData[className][studentName][dateStr] = task.completed ? "✅ Present" : "❌ Absent";
        }
    }
}

// 3. Render a separate table with calculations and headcount for each class period
let classesFound = Object.keys(masterData).sort();

if (classesFound.length === 0) {
    dv.paragraph("*No attendance data recorded in the last 7 days.*");
} else {
    for (let className of classesFound) {
        // Render Class Heading
        dv.header(2, `${className}`);
        
        let students = Object.keys(masterData[className]).sort();
        let totalStudents = students.length; 
        
        // Render Total Student Headcount right below the header
        dv.paragraph(`👥 **Total Students:** ${totalStudents}`);
        
        let sortedDates = Array.from(datesByClass[className]).sort();
        let totalSessions = sortedDates.length;
        let tableRows = [];
        
        for (let student of students) {
            let row = [student];
            let presentCount = 0;
            
            for (let date of sortedDates) {
                let status = masterData[className][student][date];
                row.push(status || "—");
                
                if (status === "✅ Present") {
                    presentCount++;
                }
            }
            
            // Calculate percentage
            let percentage = totalSessions > 0 ? Math.round((presentCount / totalSessions) * 100) : 0;
            let rateText = `**${percentage}%** (${presentCount}/${totalSessions})`;
            
            // Apply conditional formatting: Highlight red if below 70%
            if (percentage < 70) {
                row.push(`<span style="color: #ff5555; font-weight: bold;">⚠️ ${percentage}% (${presentCount}/${totalSessions})</span>`);
            } else {
                row.push(rateText);
            }
            
            tableRows.push(row);
        }
        
        // Render table
        dv.table(["Student Name", ...sortedDates, "Attendance Rate"], tableRows);
    }
}
```
