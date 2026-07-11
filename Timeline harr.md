<%*
const selectedText = tp.file.selection();

if (!selectedText) {
    new Notice("Please select the text you want to format first!");
    return "";
}

// Split the highlighted text by line breaks
const lines = selectedText.split('\n');

// Start the callout with the specific identifier from your CSS
let timelineOutput = "> [!timeline-harr]\n";

// Prepend the blockquote syntax to every selected line
for (let line of lines) {
    timelineOutput += `> ${line}\n`;
}

return timelineOutput;
%>