# Rating System For Zammad + Grafana Visualization
Manual — how to create a rating system in Zammad in conjunction with Grafana + statistics

## 1. Steps on the Zammad side 
### To create a rating system, you first need to create a new object in Zammad. <br>
To do this, go to Manage -> System -> Objects -> New attribute in Zammad: <br>
<br>
<img width="1650" height="998" alt="image" src="https://github.com/user-attachments/assets/a02db43f-3995-4ea9-a911-3592951a10fa" /> <br>

In the window that opens, fill in the fields in the following format, as shown in the photo:

<img width="320" height="764" alt="image" src="https://github.com/user-attachments/assets/90355d7a-176d-47e4-a276-8106d68a5203" /> <br>
_If you wish, you can change ‘Display’ as you see fit. (Instead of ⭐ Write good/excellent)_ <br> <br>

### Now let's create two workflows for the rating system to function correctly. <br>
To create Workflows, go to Manage -> System -> Core Workflows -> New Workflow:

<img width="1919" height="1009" alt="image" src="https://github.com/user-attachments/assets/a16ff2d8-f164-4765-bfec-6f2d096a9683" /> <br>



1. Let's start by prohibiting agents from changing ratings. Only customers should be able to change ratings! (Otherwise, it's not fair.): <br>
   Give it any name you like. Then configure the workflow as I have done: <br>

   <img width="886" height="944" alt="image" src="https://github.com/user-attachments/assets/0d2fc40f-1c8c-470e-a5d0-c31c6cee49d4" /> <br>

   _The purpose of this workflow is that if the user is an Admin/Agent, the ‘Rate’ object should be read-only._ <br> <br>

2. Now let's move on to creating the second workflow.
   <br>Its purpose is to display the rating selection field only when the ticket status is closed.

   <img width="881" height="942" alt="image" src="https://github.com/user-attachments/assets/9c98f5cc-53db-42c0-8fe5-515329114a42" /> <br>

3. (<ins>Optional</ins>) You can add a trigger that will send a message to the ticket chat asking you to rate the agent. <br>

   <img width="1686" height="1008" alt="image" src="https://github.com/user-attachments/assets/8900f397-0955-4890-8136-2b3f49413f27" /> <br>

### Result:

<img width="1686" height="954" alt="image" src="https://github.com/user-attachments/assets/bf4023da-2dcf-4390-9e26-468374819187" />

## 2. Steps on the Grafana side
### Now it's time to configure the panel in Grafana to display this rating. <br>
I won't go into detail about the Grafana web interface, as I did with Zammad. Instead, I'll just show you how to set up a panel to display the rating for each agent. <br>

1. ### Queries
   Create an empty panel on your dashboard and give it a name of your choice. In ‘Visualisation’, I recommend selecting ‘Gauge’. <br>
   Set the rest of the settings exactly as I have done: <br>

   <img width="1901" height="985" alt="image" src="https://github.com/user-attachments/assets/2b62788a-b26e-4b86-860e-f9322d39fcc1" /> <br>
   Lucene Query: state.name.keyword:"closed" AND NOT owner.fullname.keyword:"-" AND owner.fullname.keyword:"Agent Name" AND NOT rate.keyword:"" <br>
   ### Explanations:
    A. **Lucene Query** <br>
    > state.name.keyword:"closed" — filters tickets by status = closed <br>

    > NOT owner.fullname.keyword:"-" — ignores tickets that do not have an owner (agent) <br>
   
    > owner.fullname.keyword:"Agent Name" — this filter can be used to specify whose rating to display (for example, if you write John Marston in quotation marks, Grafana will only show us his rating) <br>
     
    > NOT rate.keyword:"" — does not count tickets that do not yet have a rating <br>

   B. **Group By/Then By** <br>
   > Groups the column by agent names (owner.fullname.keyword) and by rating (rate.keyword) <br>
   
**And what do we have in the end:**
The panel shows only those tickets that have a closed status, are assigned to an agent (example: John Marston) and does not show tickets that do not have a rating

2. ### Transformations
   To find an agent's rating, you need to perform a simple mathematical function — find the **arithmetic mean** of all ticket ratings (_which, of course, are specific to a given agent_). <br>
   This is the most difficult part, as Grafana displays multiple tickets with the same rating (for example, 5⭐) as one. <br>
   In any case, just follow my lead, because I've found a solution.<br>

   A. Add "Convert field type" Transformation. Convert count and rate.keyword fielts as a number: <br>
      <img width="528" height="157" alt="image" src="https://github.com/user-attachments/assets/f0639d6b-3c00-43ef-b162-46b7824f4311" /> <br>
   
   B. Add "Add field from calculation" Transformation. Repeat all settings exactly as shown in the photo: <br>
      <img width="703" height="181" alt="image" src="https://github.com/user-attachments/assets/95218092-9acb-4eb8-ad52-7ef62c397e8d" /> <br>

   C. Add "Group by" Transformation with my settings: <br>
      <img width="772" height="219" alt="image" src="https://github.com/user-attachments/assets/f1275a12-9ba5-462e-a845-e535aedcd929" /> <br>
      
   D. Add one more "Add field from calculation" Transformation with my settings: <br>
      <img width="720" height="182" alt="image" src="https://github.com/user-attachments/assets/7fa0e703-e786-48ff-9a91-bfef8de4dbaa" /> <br>

   E. And finally, let's hide the fields we don't need. Add "Organize fields by name" Transformation with my settings: <br>
      <img width="918" height="222" alt="image" src="https://github.com/user-attachments/assets/77c75e0f-fcb4-40c6-b06d-26e3f7705784" /> <br>

## That's it. Your rating system is ready!

<img width="1885" height="416" alt="image" src="https://github.com/user-attachments/assets/d861f0ef-d650-4142-9d0a-72ec4e9bf9aa" />


### It could have been better, but since I couldn't find any guides on the internet, I had to create something new.

Tip: If you have more than one agent (which is most likely the case), duplicate the panel you created earlier. In the new panel, simply change the ‘Lucene Query’ field in “Queries” to owner.fullname.keyword:‘Second Agent’. <br> <br>
In other words, in my solution, you will have to manually add a rating panel for each agent (but it only takes 15 seconds).  <br>
   
_Just in case, I will leave the JSON file for the entire dashboard (with three panels) and the JSON file for one separate panel._ <br>
1. [Panel.json file](onlyPanel.json)
2. [Dashboard.json file](dashboard.json)


**I would be grateful if you would rate my work and click on Star.**
