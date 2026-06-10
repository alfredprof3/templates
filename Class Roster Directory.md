
```dataviewjs
const storagePath = "Student-Rosters.json";
const fileExists = app.vault.getAbstractFileByPath(storagePath);

if (!fileExists) {
    dv.paragraph("⚠️ *No database found yet. Run your Roster Manager template first!*");
} else {
    let content = await app.vault.adapter.read(storagePath);
    let db = JSON.parse(content);
    
    let rawFolder = dv.current().file.folder.split("/").pop();
    let cleanedFolder = rawFolder.replace(/^[\d\s\-_]+/, "").toLowerCase().trim();
    
    let databaseClasses = Object.keys(db.classes || {});
    let matchedClassName = databaseClasses.find(cName => 
        cName.replace(/^[\d\s\-_]+/, "").toLowerCase().trim() === cleanedFolder
    );
    
    let classInfo = db.classes[matchedClassName];
    
    if (!classInfo) {
        dv.paragraph(`⚠️ *No database entry found matching cleaned folder name: "**${cleanedFolder}**" (from folder: "${rawFolder}").*`);
    } else {
        dv.header(1, `📋 ${matchedClassName}`);
        dv.paragraph(`🏫 **Institution:** ${classInfo.university} | 🗓️ **Semester Level:** ${classInfo.semester}`);
        
        const bypassLabels = ["na", "n/a", "none", "no email", "not applicable"];
        let rows = [];
        
        for (let studentId of classInfo.studentIds) {
            let student = db.students[studentId];
            if (student) {
                let emailValue = student.email ? student.email.trim() : "N/A";
                let emailDisplay = bypassLabels.includes(emailValue.toLowerCase()) 
                    ? `*${emailValue.toUpperCase()}*` 
                    : `[${emailValue}](mailto:${emailValue})`;
                    
                rows.push([student.name, `\`${studentId}\``, emailDisplay]);
            }
        }
        
        rows.sort((a, b) => a[0].localeCompare(b[0]));

        // FIXED: Print live total class headcount badge right above the table canvas layout
        dv.paragraph(`👥 **Total Registered Students:** ${rows.length}`);

        dv.table(["Student Name", "Enrollment ID", "Email Address"], rows);
    }
}
```
