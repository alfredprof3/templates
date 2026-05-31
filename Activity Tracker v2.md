# <% tp.date.now("DD-MMM-YYYY, dddd") %>
<%*
const storagePath = "Student-Rosters.json";
if (!(await tp.file.exists(storagePath))) {
    new Notice("Database not found! Run Roster Manager first."); return;
}
let content = await app.vault.adapter.read(storagePath);
let db = JSON.parse(content);
let classes = Object.keys(db.classes || {}).sort();

let chosenClass = await tp.system.suggester(classes, classes);
if (chosenClass) {
    let activityName = await tp.system.prompt("Enter the Activity Title:");
    activityName = activityName ? activityName.trim() : "Unnamed Activity";
    
    tR += `## ${chosenClass}\n📝 **Activity:** ${activityName}\n`;
    let studentIds = db.classes[chosenClass].studentIds;
    
    if (studentIds.length > 0) {
        let studentNames = studentIds.map(id => db.students[id].name).sort();
        let checklist = studentNames
            .map(name => `- [ ] ${name} (student:: ${name}) (class:: ${chosenClass}) (activity:: ${activityName})`)
            .join('\n');
        tR += checklist;
    } else {
        tR += "_No students assigned to this class section._\n";
    }
}
-%>
