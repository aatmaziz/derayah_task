Overview
You are tasked with designing and partially implementing a Real-Time Order Status and Trade Notifications feature for a multi-platform trading application (Mobile and Web).
Business Prompt:
"We want users to instantly see updates on their open orders and executions without refreshing manually."
This case study assesses your ability to work through vague requirements, architect clean scalable systems, implement critical components, and mentor less experienced engineers.

🌆 Scenario
You are the Staff Engineer responsible for delivering this cross-platform capability. Users must receive updates instantly as their orders change state (submitted, filled, partially filled, cancelled, etc.) across stocks, mutual funds, ETFs, crypto, etc.
You have full control over tech choices, but backend services and trading execution platforms already exist.

1. Problem Analysis.
   
   Clarify assumptions and open questions.
   - I assume there are existing backend services responsible for user login and authintication.
   - I assume there are a CURD opertation for create , modifiy and cancel order.
   - I assume there is a service responsible for geting orders from excution plattform.
   - I assume there is a platform type to know this user from mobile of web.
   - I assume that all integrated plattform will transform to the same internal model of order.
   - what is an interval for updates orders ?
   - what is the DB type ? 
   - what is the number of users ?
   - what is the number of orders ?
   - Is there is an agreegation for orders or user see the orders status update at the same time ?
     
   Identify what makes the requirement vague
   - "without refresh" means that it's happened with real time connection but didn't mentioned what lib to use.
   - what is the acceptable latency for this requirment.
   - the specific details of the notifications (content) are not fully provided.
     
   Define MVP scope vs full solution
   
   first mvp :
   - WebSocket integration.
   - Handles order statuses: submitted, partially filled, filled, cancelled, rejected.
   - Basic error handling and show error msg for users.
   - set up our Ui screen for getting orders list with paging.
   - ui indicators for “connection lost” and auto-reconnect.
     
   full solution :
   - latency-optimized.
   - handle fallback if socket is down.
   - handle background thread for updating data.
   - return timestamps allow the client to show precise execution timing (“filled 2 seconds ago”).
  
   Include user journeys and edge cases
   - user login to our mobile app or web.
   - user open stock details screen.
   - start creating order.
   - order is submitted and navigation to orders history screen.
   - user see order update to Partially Filled.
   - user see order update again to Filled.
   edge cases
   - lost connection need to fallback polling every X second.
   - user logs in on two devices both receive updates or logout from first device.
   - rejected or canceled orders user must error messages that order is cancel or rejected.














