```dataviewjs
// 1. Get every page that actually contains attendance tasks (student + class fields).
//    NOTE: we deliberately do NOT filter by page.file.ctime anymore (see explanation
//    in the chat reply) — for a single continuously-growing note, ctime is fixed at
//    whenever the file was first created, so it either silently excludes the whole
//    file once it's >90 days old, or does nothing useful. It's not a valid stand-in
//    for "when was this specific attendance entry taken."
let pages = dv.pages()
    .filter(p => p.file.tasks.filter(t => t.student && t.class).length > 0);

// 2. Gather attendance tasks and group them by Class -> Student -> Date
let masterData = {};
let datesByClass = {};

for (let page of pages) {
    let tasks = page.file.tasks.filter(t => t.student && t.class);

    // Read the note's raw text once per page so we can look upward from each
    // task and find the actual "# DD-MMM-YYYY, dddd" heading it sits under.
    let content = await dv.io.load(page.file.path);
    let lines = content.split("\n");

    for (let task of tasks) {
        let dateStr = null;

        // Walk backwards from the task's own line until we hit a TOP-LEVEL
        // heading ("# ..."). We intentionally skip "## Class Name" headings
        // (and anything deeper) so we land on the date, not the subject.
        for (let i = task.line; i >= 0; i--) {
            let line = lines[i];
            if (/^#\s+[^#\s]/.test(line)) {
                let match = line.match(/\d{1,2}-[A-Za-z]{3,9}-\d{4}/);
                dateStr = match ? match[0] : line.replace(/^#\s*/, "").trim();
                break;
            }
        }
        if (!dateStr) dateStr = "Unknown Date";

        let className = task.class;
        let studentName = task.student;

        if (!masterData[className]) masterData[className] = {};
        if (!masterData[className][studentName]) masterData[className][studentName] = {};
        if (!datesByClass[className]) datesByClass[className] = new Set();

        datesByClass[className].add(dateStr);
        masterData[className][studentName][dateStr] = task.completed ? "✅ Present" : "❌ Absent";
    }
}

// 3. Render a separate table with calculations and headcount for each class period
let classesFound = Object.keys(masterData).sort();

if (classesFound.length === 0) {
    dv.paragraph("*No attendance data recorded yet.*");
} else {
    for (let className of classesFound) {
        dv.header(2, `${className}`);

        let students = Object.keys(masterData[className]).sort();
        let totalStudents = students.length;

        dv.paragraph(`👥 **Total Students:** ${totalStudents}`);

        // Sort dates chronologically, not alphabetically — "DD-MMM-YYYY" strings
        // don't sort correctly as plain text (e.g. "Aug" would sort before "Jan").
        let sortedDates = Array.from(datesByClass[className]).sort((a, b) => {
            let ma = moment(a, "DD-MMM-YYYY");
            let mb = moment(b, "DD-MMM-YYYY");
            if (!ma.isValid() && !mb.isValid()) return 0;
            if (!ma.isValid()) return 1;
            if (!mb.isValid()) return -1;
            return ma.valueOf() - mb.valueOf();
        });
        let totalSessions = sortedDates.length;
        let tableRows = [];

        for (let student of students) {
            let row = [student];
            let presentCount = 0;

            for (let date of sortedDates) {
                let status = masterData[className][student][date];
                row.push(status || "—");
                if (status === "✅ Present") presentCount++;
            }

            let percentage = totalSessions > 0 ? Math.round((presentCount / totalSessions) * 100) : 0;
            let rateText = `**${percentage}%** (${presentCount}/${totalSessions})`;

            if (percentage < 70) {
                row.push(`<span style="color: #ff5555; font-weight: bold;">⚠️ ${percentage}% (${presentCount}/${totalSessions})</span>`);
            } else {
                row.push(rateText);
            }

            tableRows.push(row);
        }

        dv.table(["Student Name", ...sortedDates, "Attendance Rate"], tableRows);
    }
}
```