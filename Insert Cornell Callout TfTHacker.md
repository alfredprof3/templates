
<%*
// Prompt to input the Title
let title = await tp.system.prompt("Callout Title");
%>
> [!cue] <% title %>
> <% tp.file.cursor(1) %>
