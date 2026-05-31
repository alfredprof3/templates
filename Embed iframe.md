
<% tp.file.cursor(2) %>
<%*
// Prompt to input the Title
let title = await tp.system.prompt("Enter Title");
// Prompt to input the url
let url = await tp.system.prompt("Enter URL");
%>
[<% title %>](<% url %>)
<% tp.file.cursor(1) %>\<iframe src="<% url %>" title="<% title %>" width="100%" height="800px" scrolling="no" frameborder="no" allow="fullscreen"></iframe>
