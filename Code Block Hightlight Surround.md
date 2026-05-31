
<%*
// 1. Get any text you currently have highlighted
let selection = tp.file.selection();

// 2. Define the list of programming languages you want in your dropdown menu
let languages = ["javascript", "bash", "markdown", "python", "css", "html", "json", "typescript", "sql"];

// 3. Display the dropdown menu
let chosenLanguage = await tp.system.suggester(languages, languages);

// 4. Fallback default language if you accidentally hit Escape/cancel the menu
if (!chosenLanguage) {
    chosenLanguage = "";
}

// 5. Output the formatted Markdown code block
if (selection) {
    // If you highlighted text, wrap it cleanly
    tR = `\`\`\`${chosenLanguage}\n${selection}\n\`\`\``;
} else {
    // If no text was highlighted, leave the cursor inside an empty code block
    tR = `\`\`\`${chosenLanguage}\n\n\`\`\``;
}
-%>
