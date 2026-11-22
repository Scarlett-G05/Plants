Plant Shelf

Explaination:
Plant Shelf is a web application for swapping plants. The basic idea is an organiser hosts the website and enters in people's plants to its database. Then, during a plant swap meeting they exchange plants. On plantswap this is represented by "swapped" status. Plants can be added, deleted, swapped and un-swapped. Each plant has a name, a spcies and a light level.

How to run:
Start the express server- open terminal. Enter "cd S2bStarterExpress" then enter "npm start".
Start the svelte app- open terminal. Enter "cd S2bStarterSveltekit" then enter "npm run dev"
The terminal will dsplay the ports you need to search to open. local host 3001 for express server and local host 5173 for svelte app.

Process:
a. user enters name, species and selects light level from drop down menu
b. the svelte plant API sends the form data to the express server. This is done via a post request.
c. express saves the plant data
d. mongoose stores the plant on MongoDB
e. the new plant is displayed on the shelf