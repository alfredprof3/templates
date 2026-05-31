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
    // 2. Sub-menu for editing type
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
        
        // UPGRADED: Prompt to specifically edit the Class/Course Title
        let newClassName = await tp.system.prompt("Edit Course Name (Title):", chosenClass);
        let newUniv = await tp.system.prompt("Edit College/University Name:", classObj.university);
        let newSem = await tp.system.prompt("Edit Semester/Grade Level:", classObj.semester);
        
        if (newClassName !== null && newUniv !== null && newSem !== null) {
            newClassName = newClassName.trim();
            
            // If the course name changed, clone the data to the new key and delete the old one
            if (chosenClass !== newClassName) {
                db.classes[newClassName] = {
                    university: newUniv.trim(),
                    semester: newSem.trim(),
                    studentIds: classObj.studentIds
                };
                delete db.classes[chosenClass];
                newNotice(`Renamed class from ${chosenClass} to ${newClassName}`);
            } else {
                // If the name stayed the same, just update university and semester fields
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
        
        className = await tp.system.prompt("Enter NEW Course Name:", baseClass);
        defaultUniv = db.classes[baseClass].university;
        defaultSem = db.classes[baseClass].semester;
        existingIds = [...db.classes[baseClass].studentIds];
    } else {
        className = await tp.system.prompt("Enter New Course Name (e.g., Marketing):");
    }
    
    if (!className) return;
    className = className.trim();
    
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
            if (chosenId && !studentIds.includes(chosenId)) {
                studentIds.push(chosenId);
                new Notice(`Added ${db.students[chosenId].name} to ${className}.`);
            }
        } else if (subMode === "brand_new") {
            let sId = await tp.system.prompt("Enter Student Enrollment ID:");
            if (!sId) continue;
            sId = sId.trim();
            
            let sName = await tp.system.prompt("Enter Student Name:");
            let sEmail = await tp.system.prompt("Enter Student Email:");
            
            db.students[sId] = { name: sName.trim(), email: sEmail.trim() };
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
