
<%*
let selection = tp.file.selection();

let options = ["You", "Claude", "ChatGPT", "Gemini", "DeepSeek", "Copilot", "Qwen", "AI Mode by Google"];
let chosen = await tp.system.suggester(options, options);

if (!chosen) {
    chosen = "You";
}

let calloutType = (chosen === "You") ? "human" : "ai";

let processedText = "";
if (selection) {
    processedText = selection
        .split('\n')
        .map(line => `> ${line}`)
        .join('\n');
} else {
    processedText = "> ";
}

tR = `> [!${calloutType}] ${chosen}\n${processedText}`;
-%>
