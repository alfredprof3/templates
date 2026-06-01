<% tp.file.cursor(1) %>
<%*
// Prompt to input the Title
let title = await tp.system.prompt("Enter Title");
// Prompt to input the url
let url = await tp.system.prompt("Enter URL");
%>
# <% title %>
![<% title %>](<% url %>)
