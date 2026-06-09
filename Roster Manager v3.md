<%*
const storagePath = "Student-Rosters.json";
let db = { students: {}, classes: {} };

if (await tp.file.exists(storagePath)) {
    let content = await app.vault.adapter.read(storagePath);
    try { db = JSON.parse(content); } catch(e) {}
}
if (!db.students) db.students = {};
if (!db.classes) db.classes = {};

// 1. Main Menu Selector
let mode = await tp.system.suggester(
    ["➕ Create a Brand New Class", "🔄 Rollover/Modify Existing Class (New Semester/Course)", "✏️ Edit Student Info or Class Metadata", "❌ Remove Student from a Class", "🗑️ Delete a Class Database"],
    ["new", "modify", "edit_info", "remove_student", "delete_class"]
);

if (mode === "edit_info") {
    let editType = await tp.system.suggester(
        ["👤 Edit a Student's Profile (Name, Email, or ID)", "🏫 Edit Class Metadata (Course Name, University, or Semester)"],
        ["student", "class_meta"]
    );
    
    if (editType === "student") {
        let globalIds = Object.keys(db.students);
        if (globalIds.length === 0) { new Notice("Global database is empty!"); return; }
        
        let globalNames = globalIds.map(id => `${db.students[id].name} (${id})`);
        let oldId = await tp.system.suggester(globalNames, globalIds);
        if (!oldId) return;
        
        let studentData = db.students[oldId];
        
        let newName = await tp.system.prompt("Edit Student Name:", studentData.name);
        let newEmail = await tp.system.prompt("Edit Student Email:", studentData.email);
        let newId = await tp.system.prompt("Edit Student ID (Warning: Changes ID everywhere):", oldId);
        
        if (!newName || !newEmail || !newId) { new Notice("Edits canceled. Fields cannot be empty."); return; }
        
        newName = newName.trim();
        newEmail = newEmail.trim();
        newId = newId.trim();
        
        if (oldId !== newId && db.students[newId]) {
            new Notice(`⚠️ ERROR: The ID '${newId}' belongs to another student!`);
            return;
        }
        
        if (oldId !== newId) {
            delete db.students[oldId];
            for (let cName in db.classes) {
                db.classes[cName].studentIds = db.classes[cName].studentIds.map(id => id === oldId ? newId : id);
            }
        }
        
        db.students[newId] = { name: newName, email: newEmail };
        await app.vault.adapter.write(storagePath, JSON.stringify(db, null, 2));
        new Notice(`Successfully updated profile for ${newName}!`);
        
    } else if (editType === "class_meta") {
        let classes = Object.keys(db.classes);
        if (classes.length === 0) { new Notice("No classes exist."); return; }
        
        let chosenClass = await tp.system.suggester(classes, classes);
        if (!chosenClass) return;
        
        let classObj = db.classes[chosenClass];
        let newClassName = await tp.system.prompt("Edit Course Name (Title):", chosenClass);
        let newUniv = await tp.system.prompt("Edit College/University Name:", classObj.university);
        let newSem = await tp.system.prompt("Edit Semester/Grade Level:", classObj.semester);
        
        if (newClassName !== null && newUniv !== null && newSem !== null) {
            newClassName = newClassName.trim();
            
            if (chosenClass !== newClassName) {
                if (db.classes[newClassName]) {
                    new Notice(`⚠️ ERROR: A class named '${newClassName}' already exists.`);
                    return;
                }
                db.classes[newClassName] = {
                    university: newUniv.trim(),
                    semester: newSem.trim(),
                    studentIds: classObj.studentIds
                };
                delete db.classes[chosenClass];
            } else {
                db.classes[chosenClass].university = newUniv.trim();
                db.classes[chosenClass].semester = newSem.trim();
            }
            
            await app.vault.adapter.write(storagePath, JSON.stringify(db, null, 2));
            new Notice(`Updated class details successfully.`);
        }
    }

} else if (mode === "new" || mode === "modify") {
    let className = "";
    let defaultUniv = "", defaultSem = "", existingIds = [];
    
    if (mode === "modify") {
        let classes = Object.keys(db.classes);
        if (classes.length === 0) { new Notice("No classes exist to modify."); return; }
        let baseClass = await tp.system.suggester(classes, classes);
        if (!baseClass) return;
        
        className = baseClass;
        defaultUniv = db.classes[baseClass].university;
        defaultSem = db.classes[baseClass].semester;
        existingIds = [...db.classes[baseClass].studentIds];
    } else {
        className = await tp.system.prompt("Enter New Course Name (e.g., Marketing):");
        if (!className) return;
        className = className.trim();
        if (db.classes[className]) {
            new Notice(`⚠️ ERROR: Class '${className}' already exists!`);
            return;
        }
    }
    
    let university = await tp.system.prompt("Enter College/University Name:", defaultUniv);
    let semester = await tp.system.prompt("Enter Semester/Grade Level:", defaultSem);
    
    let studentIds = [...existingIds];
    let adding = true;
    
    while (adding) {
        let subMode = await tp.system.suggester(
            ["👥 Add student from Global Database", "🆕 Register a completely brand new student", "✅ Done managing student list"],
            ["global", "brand_new", "done"]
        );
        
        if (subMode === "global") {
            let globalIds = Object.keys(db.students);
            let globalNames = globalIds.map(id => `${db.students[id].name} (${id})`);
            if (globalIds.length === 0) {
                new Notice("Global database is empty! Register a brand new student first.");
                continue;
            }
            let chosenId = await tp.system.suggester(globalNames, globalIds);
            if (chosenId) {
                if (studentIds.includes(chosenId)) {
                    new Notice(`⚠️ ALERT: ${db.students[chosenId].name} is already added to this class!`);
                } else {
                    studentIds.push(chosenId);
                    new Notice(`Added ${db.students[chosenId].name} to ${className}.`);
                }
            }
        } else if (subMode === "brand_new") {
            let sId = await tp.system.prompt("Enter Student Enrollment ID:");
            if (!sId) continue;
            sId = sId.trim();
            
            // Check ID Duplicate
            if (db.students[sId]) {
                let existingStudent = db.students[sId];
                new Notice(`⚠️ DUPLICATE ID FOUND: Assigned to ${existingStudent.name}`);
                
                let duplicateChoice = await tp.system.suggester(
                    [`🔗 Add existing student (${existingStudent.name}) to this class`, `✏️ Overwrite global info for ID ${sId}`, `❌ Cancel entry`],
                    ["add_existing", "overwrite", "cancel"]
                );
                
                if (duplicateChoice === "add_existing") {
                    if (!studentIds.includes(sId)) {
                        studentIds.push(sId);
                        new Notice(`Added ${existingStudent.name} to ${className}.`);
                    } else {
                        new Notice("⚠️ Student is already in this class list.");
                    }
                    continue;
                } else if (duplicateChoice === "overwrite") {
                    let sName = await tp.system.prompt("Enter New Student Name:", existingStudent.name);
                    let sEmail = await tp.system.prompt("Enter New Student Email:", existingStudent.email);
                    db.students[sId] = { name: sName.trim(), email: sEmail.trim() };
                    if (!studentIds.includes(sId)) studentIds.push(sId);
                    new Notice(`Profile updated and added.`);
                    continue;
                } else {
                    continue;
                }
            }
            
            let sName = await tp.system.prompt("Enter Student Name:");
            if (!sName) continue;
            sName = sName.trim();
            
            // Scan database for duplicate names
            let existingWithSameName = Object.keys(db.students).find(id => db.students[id].name.toLowerCase() === sName.toLowerCase());
            if (existingWithSameName) {
                new Notice(`⚠️ WARNING: A student named '${sName}' is already registered with ID: ${existingWithSameName}`);
                let choiceName = await tp.system.suggester(
                    [`🔗 Link existing student (${sName} - ID: ${existingWithSameName}) to this class`, `🆕 Ignore and continue creating a distinct student`],
                    ["link", "continue"]
                );
                if (choiceName === "link") {
                    if (!studentIds.includes(existingWithSameName)) {
                        studentIds.push(existingWithSameName);
                        new Notice(`Added ${sName} to class.`);
                    } else {
                        new Notice("⚠️ Student is already in this class list.");
                    }
                    continue;
                }
            }
            
            let sEmail = await tp.system.prompt("Enter Student Email (or type NA if none):");
            if (!sEmail) continue;
            sEmail = sEmail.trim();
            
            // FIXED: Define non-applicable placeholder labels
            const bypassLabels = ["na", "n/a", "none", "no email", "not applicable"];
            const isBypassEmail = bypassLabels.includes(sEmail.toLowerCase());

            // FIXED: Only scan for duplicate emails if it is NOT a bypass label
            if (!isBypassEmail) {
let existingWithSameEmail = Object.keys(db.students).find(id => db.students[id].email.toLowerCase() === sEmail.toLowerCase());  
if (existingWithSameEmail) {  
new Notice(`⚠️ WARNING: The email '${sEmail}' is already assigned to ${db.students[existingWithSameEmail].name}`);  
let choiceEmail = await tp.system.suggester(  
[`🔗 Link existing student (${db.students[existingWithSameEmail].name}) to this class`, `❌ Cancel and re-enter`],  
["link", "cancel"]  
);  
if (choiceEmail === "link") {  
if (!studentIds.includes(existingWithSameEmail)) {  
studentIds.push(existingWithSameEmail);  
new Notice(`Added ${db.students[existingWithSameEmail].name} to class.`);  
} else {  
new Notice("⚠️ Student is already in this class list.");  
}  
continue;  
} else {  
continue;  
}  
}  
}

// Save student profile  
db.students[sId] = { name: sName, email: sEmail };  
studentIds.push(sId);  
new Notice(`Registered and added ${sName}.`);  
} else {  
adding = false;  
}  
}

db.classes[className] = { university: university.trim(), semester: semester.trim(), studentIds: studentIds };  
await app.vault.adapter.write(storagePath, JSON.stringify(db, null, 2));  
new Notice(`Class ${className} successfully saved!`);

} else if (mode === "remove_student") {  
let classes = Object.keys(db.classes);  
let chosenClass = await tp.system.suggester(classes, classes);  
if (!chosenClass) return;

let currentIds = db.classes[chosenClass].studentIds;  
let names = currentIds.map(id => db.students[id]?.name || id);

let toRemove = await tp.system.suggester(names, currentIds);  
if (toRemove) {  
db.classes[chosenClass].studentIds = currentIds.filter(id => id !== toRemove);  
await app.vault.adapter.write(storagePath, JSON.stringify(db, null, 2));  
new Notice(`Removed student from ${chosenClass}.`);  
}  
} else if (mode === "delete_class") {  
let classes = Object.keys(db.classes);  
let chosenClass = await tp.system.suggester(classes, classes);  
if (chosenClass) {  
let confirm = await tp.system.suggester(["❌ Cancel", "🗑️ Confirm Delete Class"], [false, true]);  
if (confirm) {  
delete db.classes[chosenClass];  
await app.vault.adapter.write(storagePath, JSON.stringify(db, null, 2));  
new Notice(`${chosenClass} removed from database.`);  
}  
}  
}  
-%>