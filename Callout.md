<%*
let labels = ["Note","Info","Tip","Check","Warning","Danger","Bug","Question","Abstract","Example","Quote","Failure"];
let values = ["note","info","tip","check","warning","danger","bug","question","abstract","example","quote","failure"];

let chosen = await tp.system.suggester(labels, values);

if (chosen) {
    let title = "";
    try {
        let t = await tp.system.prompt("Title (Enter to skip)", "");
        if (t) { title = t.trim(); }
    } catch(e) {}

    let ed = null;
    let selection = "";
    try {
        ed = app.workspace.activeEditor.editor;
        selection = ed.getSelection() || "";
    } catch(e) {
        selection = tp.file.selection() || "";
    }

    let header = "> [!" + chosen + "]";
    if (title !== "") { header += " " + title; }

    let content = selection.trim() !== ""
        ? selection.split("\n").map(function(l) { return "> " + l; }).join("\n")
        : "> ";

    let result = header + "\n" + content;

    if (ed) {
        ed.replaceSelection(result);
        tR = "";
    } else {
        tR = result;
    }
}
%>