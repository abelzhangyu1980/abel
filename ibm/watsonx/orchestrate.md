# automation framework
  to connect to existing apps via api calls.
  connector, team, apps, skills. skill flow
# conversational framework
  ai powered framework to allow conversational way to interact with apps.


watsonx ADK

create tool(s)
   tools act as the bridge between the orchestrate and the implementation (basically a wrapper which knows how to connect to the service and how to call the service).
create agent
   references to tools
   define which llm model to use
   behaviour -- prompt --- tell which tool to be used for the conversational text. use llm to extract the data from human words and then feed the data into tools.
  
how to setup connections to external applications
 wrapper + config + credentials

 via command line or gui

 then ask ai to create the code for the tool....

 how to install and configure ADK

 then add your wation x server to the adk env.
 activate the env --- your api key 
 
 
 so basically we install and configure ADK on local and then use command lines to create agents/tools...
 
