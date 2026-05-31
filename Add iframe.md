
<% tp.file.cursor(2) %>
<%*
// Prompt to input the url
let url = await tp.system.prompt("Enter URL");
%>
<% tp.file.cursor(1) %>\<iframe src="<% url %>" width="100%" height="800px" scrolling="no" frameborder="no" allow="fullscreen"></iframe>
