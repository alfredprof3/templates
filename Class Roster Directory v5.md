
```dataviewjs
const storagePath = "Student-Rosters.json";
const fileExists = app.vault.getAbstractFileByPath(storagePath);

if (!fileExists) {
    dv.paragraph("⚠️ *No database found yet. Run your Roster Manager template first!*");
} else {
    let content = await app.vault.adapter.read(storagePath);
    let db = JSON.parse(content);
    
    let rawFolder = dv.current().file.folder.split("/").pop();
    
    // Normalize folder name: removes prefixes and handles hyphens/spaces
    let cleanedFolder = rawFolder
        .replace(/^[\d\s\-_]+/, "")
        .replace(/-/g, " ")
        .toLowerCase()
        .trim();
    
    let folderWords = cleanedFolder.split(/\s+/);
    let databaseClasses = Object.keys(db.classes || {});
    
    // Flexible keyword matching to bridge differences between folder names and ALL CAPS database keys
    let matchedClassName = databaseClasses.find(cName => {
        let cleanClassName = cName.replace(/^[\d\s\-_]+/, "").replace(/-/g, " ").toLowerCase().trim();
        return folderWords.every(word => cleanClassName.includes(word));
    });
    
    let classInfo = db.classes[matchedClassName];
    
    if (!classInfo) {
        dv.paragraph(`⚠️ *No database entry found matching cleaned folder name: "**${cleanedFolder}**" (from folder: "${rawFolder}").*`);
    } else {
        // 1. The class name
        dv.header(1, `📋 ${matchedClassName}`);
        
        // 2. The institution & 3. The semester level
        dv.paragraph(`🏫 **Institution:** ${classInfo.university} | 🗓️ **Semester Level:** ${classInfo.semester}`);
        
        const bypassLabels = ["na", "n/a", "none", "no email", "not applicable"];
        let rows = [];
        
        for (let studentId of classInfo.studentIds) {
            let student = db.students[studentId];
            if (student) {
                let emailValue = student.email ? student.email.trim() : "N/A";
                let emailDisplay = "";

                // 5. Check for "NA" labels: italicize if NA, display as plain text otherwise
                if (bypassLabels.includes(emailValue.toLowerCase())) {
                    emailDisplay = `*${emailValue.toUpperCase()}*`;
                } else {
                    emailDisplay = emailValue; // Plain text strings (No clickable markdown links)
                }
                    
                rows.push([student.name, `\`${studentId}\``, emailDisplay]);
            }
        }
        
        rows.sort((a, b) => a[0].localeCompare(b[0]));

        // 4. The total number of enrolled students
        dv.paragraph(`👥 **Total Registered Students:** ${rows.length}`);

        dv.table(["Student Name", "Enrollment ID", "Email Address"], rows);
    }
}
```
