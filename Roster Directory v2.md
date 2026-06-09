
```dataviewjs
// 1. Configuration: Path to the central database JSON file
const storagePath = "Student-Rosters.json";
const fileExists = app.vault.getAbstractFileByPath(storagePath);

if (!fileExists) {
    dv.paragraph("⚠️ *No database found yet. Run your Management Template first!*");
} else {
    // 2. Load and parse the central database
    let content = await app.vault.adapter.read(storagePath);
    let db = JSON.parse(content);
    let classes = Object.keys(db.classes || {}).sort();

    if (classes.length === 0) {
        dv.paragraph("*No active classes found.*");
    } else {
        // Define our exact bypass placeholder list matching the Roster Manager
        const bypassLabels = ["na", "n/a", "none", "no email", "not applicable"];

        // 3. Generate contact tables for each individual class period
        for (let className of classes) {
            let classInfo = db.classes[className];
            dv.header(2, `📋 ${className}`);
            dv.paragraph(`🏫 **Institution:** ${classInfo.university} | 🗓️ **Semester:** ${classInfo.semester}`);
            
            let rows = [];
            for (let studentId of classInfo.studentIds) {
                let student = db.students[studentId];
                if (student) {
                    let emailValue = student.email ? student.email.trim() : "N/A";
                    let emailDisplay = "";

                    // FIXED: Check if the email string is a standard placeholder string
                    if (bypassLabels.includes(emailValue.toLowerCase())) {
                        // Render as simple, gray italicized text indicating missing contact info
                        emailDisplay = `*${emailValue.toUpperCase()}*`;
                    } else {
                        // Render as an active, clickable direct mail link
                        emailDisplay = `[${emailValue}](mailto:${emailValue})`;
                    }

                    rows.push([
                        student.name,
                        `\`${studentId}\``,
                        emailDisplay
                    ]);
                }
            }
            
            // Sort student rows alphabetically by name
            rows.sort((a, b) => a[0].localeCompare(b[0]));
            dv.table(["Student Name", "Enrollment ID", "Email Address"], rows);
        }
    }
}
```
