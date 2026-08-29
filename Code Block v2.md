<%*
// 1. Define your programming languages here. 
// You can add, remove, or reorder these at any time.
const langs = [
  "bash",
  "css",
  "html",
  "javascript",
  "json",
  "markdown",
  "nvim",
  "python",
  "sql",
  "toml",
  "typescript",
  "yaml",
  "vim",
  "latex"
];

// 2. Prompt the user with a searchable dropdown list
const choice = await tp.system.suggester(langs, langs);

// 3. Handle the selection (if you press Escape, it defaults to no language)
const selectedLang = choice ? choice : "";
_%>
```<% selectedLang %>
<% tp.file.cursor(1) %>
```