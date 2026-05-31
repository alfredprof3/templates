
<%*
// Prompt to input the url
let url = await tp.system.prompt("Paste the code");
// Prompt to input the Title
let title = await tp.system.prompt("Enter Title");
%>
## <% title %>
<% url %>
