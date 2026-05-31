
<%*
// 1. Get the highlighted text
let selection = tp.file.selection();

// 2. Open the dropdown menu to select the callout mood/type
let options = ["Gemini", "ChatGPT", "DeepSeek", "Copilot", "Qwen", "Claude"];
let chosenMood = await tp.system.suggester(options, options);

// 3. Fallback if the user presses ESC or cancels the menu
if (!chosenMood) {
    chosenMood = "Default";
}

// 4. Format and output the text
if (selection) {
    let processedText = selection
        .split('\n')
        .map(line => `> ${line}`)
        .join('\n');
        
    // Injects the chosen menu option into the callout header
    tR = `> [!author] ${chosenMood}\n${processedText}`;
} else {
    // Fallback if no text was highlighted
    tR = `> [!author] ${chosenMood}\n> `;
}
-%>
