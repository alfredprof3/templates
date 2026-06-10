
```dataviewjs
const storagePath = "Student-Rosters.json";
const fileExists = app.vault.getAbstractFileByPath(storagePath);

if (!fileExists) {
    dv.paragraph("⚠️ *No database found yet. Run your Roster Manager template first!*");
} else {
    let content = await app.vault.adapter.read(storagePath);
    let db = JSON.parse(content);
    
    let rawUnivFolder = dv.current().file.folder.split("/").pop();
    let cleanedUnivFolder = rawUnivFolder.replace(/^[\d\s\-_]+/, "").toLowerCase().trim();
    
    dv.header(1, `🏫 ${rawUnivFolder.replace(/^[\d\s\-_]+/, "")}`);
    
    let classes = Object.keys(db.classes || {}).filter(cName => {
        let dbUnivCleaned = db.classes[cName].university.replace(/^[\d\s\-_]+/, "").toLowerCase().trim();
        return dbUnivCleaned === cleanedUnivFolder;
    }).sort();

    if (classes.length === 0) {
        dv.paragraph(`⚠️ *No data found matching university folder name: "**${cleanedUnivFolder}**".*`);
    } else {
        const bypassLabels = ["na", "n/a", "none", "no email", "not applicable"];
        
        for (let className of classes) {
            dv.header(2, `📚 ${className} (Grade Level: ${db.classes[className].semester})`);
            
            let rows = [];
            for (let studentId of db.classes[className].studentIds) {
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

            // FIXED: Print live section headcount badge right underneath each course subheader
            dv.paragraph(`👥 **Total Students in Section:** ${rows.length}`);

            dv.table(["Student Name", "Enrollment ID", "Email Address"], rows);
        }
    }
}
```
