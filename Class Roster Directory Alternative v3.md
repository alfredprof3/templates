
<%*
// 1. Define paths and read the central database file
const storagePath = "Student-Rosters.json";
const fileExists = app.vault.getAbstractFileByPath(storagePath);

if (!fileExists) {
    new Notice("⚠️ Student-Rosters.json database file could not be found!");
    return;
}

let content = await app.vault.adapter.read(storagePath);
let db = JSON.parse(content);
let databaseClasses = Object.keys(db.classes || {});

// Sort classes alphabetically for the dropdown panel
databaseClasses.sort((a, b) => a.localeCompare(b));

// 2. Open the prompt window for the user to select the group section
let selectedGroup = await tp.system.suggester(databaseClasses, databaseClasses, false, "Select the Course Group for this Roster:");

if (!selectedGroup) {
    new Notice("❌ No group selected. Template generation cancelled.");
    return;
}
-%>
```dataviewjs
// Target group hardcoded automatically by your template selection
const targetGroup = "<% selectedGroup %>";
const storagePath = "Student-Rosters.json";

let content = await app.vault.adapter.read(storagePath);
let db = JSON.parse(content);
let classInfo = db.classes[targetGroup];

if (!classInfo) {
    dv.paragraph(`⚠️ *Error: The group "${targetGroup}" could not be found in the roster database.*`);
} else {
    // 1. Display Class Name
    dv.header(1, `📋 ${targetGroup}`);
    
    // 2. Display Institution & 3. Semester Level
    dv.paragraph("🏫 **Institution:** " + classInfo.university + " | 🗓️ **Semester Level:** " + classInfo.semester);
    
    const bypassLabels = ["na", "n/a", "none", "no email", "not applicable"];
    let rows = [];
    
    for (let studentId of classInfo.studentIds) {
        let student = db.students[studentId];
        if (student) {
            let emailValue = student.email ? student.email.trim() : "N/A";
            let emailDisplay = "";

            // Plain Text formatting for active student emails
            if (bypassLabels.includes(emailValue.toLowerCase())) {
                emailDisplay = `*${emailValue.toUpperCase()}*`;
            } else {
                emailDisplay = emailValue; 
            }
                
            rows.push([student.name, "`" + studentId + "`", emailDisplay]);
        }
    }
    
    // Sort alphabetically by student name
    rows.sort((a, b) => a[0].localeCompare(b[0]));

    // 4. Display Registered Student Total 
    dv.paragraph("👥 **Total Registered Students:** " + rows.length);

    // Render clean layout canvas
    dv.table(["Student Name", "Enrollment ID", "Email Address"], rows);
}
```
