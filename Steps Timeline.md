<%*
const selectedText = tp.file.selection();

if (!selectedText) {
    new Notice("Please select an ordered list first!");
    return "";
}

// Split the selected text by line breaks
const lines = selectedText.split('\n');
let timelineOutput = "> [!steps]\n";

// Prepend the blockquote syntax to every line to build the callout
for (let line of lines) {
    timelineOutput += `> ${line}\n`;
}

return timelineOutput;
%>