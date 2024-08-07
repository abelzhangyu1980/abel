# salesforce functional capability
   ## marketing
      lead generation, drip marketing, website integration, campaign management, campaign testing, marketing automation, email marketing, mobile messaging, social media
      marketing, digital adveertising, customer interactions, customer segmentation, compaign optimization, contentm management, journey management.
   ## sales and e-commerce
      lead management, account managment, opportunity /pipeline mgmt, contact managment, configure, price, quote, territory managment, social network integration,personalization,
      ## partner managment, forecasting, high velocity sales, order management, b2b commerce, b2c commerce, product management, customer segmentation, payment, search engine 
      optimization, budget & buying authority, shipping managment,
   ## service
      case & incident mgmt, knowledge base, phone integration, live website chat, social networks integration, chatbot, omni-channel routing, macro automation,
      asset managment, field service 
   ## communities
      self service community, plus community, merchant communities, isv community, user community
   ## IoT & AI
      marketing inteligence, connected assets, large scale ingestion, rules based actions, predictive insight, 
   ## Analytics
   ## Apps Ecosystem
   ## Support

# The salesforce value chain
  prospecting            lead generation & conversion        sales                                                     back office/support


  pipeline creation      discovery & qualification           solution design/negotiation/review closed won/lost        open issue, closed issue

# Salesforce customer 360

# Typical ways into salesforce
## Salesforce Administrator
   * organizational setup
   * user setup
   * security & access
   * standard & custom objects
   * sales & marketing apps
   * service & support apps
   * activity managment & collaboration
   * data management
   * analytics - reports & dashboards
   * workflow / process automation
   * desktop & mobile administration
   * appExchange
### Advanced Administrator
* security & access
* extending custom objects & apps
* auditing & monitoring
* sales cloud apps
* service cloud apps
* data management
* content management
* change management
* analytics, reports and dashboards
* process automation
## Salesforce developer (app developer , code developer)
### app builder
#### platform app builder
* busienss logic and process automation
* data modeling and managment
* salefroce fundamentals
* security
* user interface
* reporting
* mobile
* app development
* social
## Technical Consultant
## Functional Consultant
## Business Analyst 
## Solution architect
## Technical Architect

# where should I srtart from?
## what should I start learning? 
84% sales cloud , chatter 72%  66% saleforce plateform, 57% service cloud 46% saleforce lightning

# Platform developer
## relationships

lookup, master-detail, many to many, exernal lookup, indirect lookup , hierarchical

### one to many  (parent - child)
manager --> position 
manage --> employee
school --> students
position -> application
relationship always go with child object.

### Lookup relationship vs master Detail
lookup is a loosely coupled relationship,allowing you to connect one object to another in a one-to-many fashion.
i.e a sales order object may be related to a store object. each sales order is related to one store and each store is related to one or more sales orders.
the sales order and store object can exist independently.
you can have a maximum of 40 lookups on an object.

master detail relationship is a strongly coupled relationship , meaning if the parent is deleted, so are the child objects.
master-detail also allows the parent record to control child record atributes such as sharing and visiblity. whichever security setting you chose
for the parent record, the child record inherits.
i.e room -> event. 
you can have a maximum of two master details on an object.

### many to many
many to many relationship allow two objects to be related to each other when a record from one object can be linked to multple records from another objects
and vice versa.
two master-detail relationships can be used to create a many-to-many relationship between two objects. a many-to-many relationship simply allow the master
record to have mutiple detail records and the detail records to have multiple master or parent records. 
ie. job position <-- job application --> candidate
doctor <--> patient

a junction object which has two master detail relationship

job application will be the junction object. job position (master) --- job application (detail) --- candidate(master)

### Hierarchical relationship
can be created on user object to allow relating one user to another 
- user object
- usage

## External Object
External objects are similar to custom objects except that they map to data that's stored outside of saleforce org. 
it relies on an external data source definition to connect with the external system's data.

## Validation rules
validtion rules verify that the data a user enters in a record meets the standards you specify before the user can save the record.

--if user has choosen a specific value and the other field is blank, then show a validation error.
i.e ISPICKVAL(field,specific picklist value) && ISBlank(..));

# Importing and Exporting Data
## Importing data into development environment
- data import wizard
  up to 50K records
- data loader
  cli 
  can import large files that contain up to 5m records (csv)
## Exporting data 
- data loader
- exporting data via reports

- 





   

