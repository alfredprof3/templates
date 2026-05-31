
```dataviewjs
const storagePath = "Student-Rosters.json";
const fileExists = app.vault.getAbstractFileByPath(storagePath);

if (!fileExists) {
    dv.paragraph("⚠️ *No database found yet. Run your Management Template first!*");
} else {
    let content = await app.vault.adapter.read(storagePath);
    let db = JSON.parse(content);
    let classes = Object.keys(db.classes || {}).sort();

    if (classes.length === 0) {
        dv.paragraph("*No active classes found.*");
    } else {
        for (let className of classes) {
            let classInfo = db.classes[className];
            dv.header(3, `📋 ${className}`);
            dv.paragraph(`🏫 **Institution:** ${classInfo.university} | 🗓️ **Semester:** ${classInfo.semester}`);
            
            let rows = [];
            for (let studentId of classInfo.studentIds) {
                let student = db.students[studentId];
                if (student) {
                    rows.push([
                        student.name,
                        `\`${studentId}\``,
                        `[${student.email}](mailto:${student.email})`
                    ]);
                }
            }
            
            // Sort table by student name rows
            rows.sort((a, b) => a[0].localeCompare(b[0]));
            dv.table(["Student Name", "Enrollment ID", "Email Address"], rows);
        }
    }
}
```
