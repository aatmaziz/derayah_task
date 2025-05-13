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
   - set up our Ui screen for getting orders list with paging library.
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
   - lost connection need to fallback polling every X second or show retry dianlog to pull new order updates.
   - user logs in on two devices both receive updates or logout from first device.
   - rejected or canceled orders user must error messages that order is cancel or rejected.

2.System Design Document

   - Architecture diagram attched please check diagram/.
     
     Explanation of real-time mechanism (WebSockets, FCM
   - WebSockets for live updates while app is foregrounded , FCM for mobile background delivery will explain it in review.
     
     Platform-specific data flow (Mobile)
   - User action is sent to the backend services via REST APIs.
     Backend services process the request and update the database.
     Upon a relevant event, the Real-time Notification Service identifies the user's mobile devices.
     Foreground: If the app has an active WebSocket connection, the update is sent via WebSocket.
     Background: If the app is in the background, the Real-time Notification Service sends a push notification (via FCM) to the user's device containing a brief summary of the update.
     When the user opens the app from the push notification (or directly), the app can either:
     Establish a WebSocket connection to receive further real-time updates.
     Fetch the latest order and trade information via REST API calls.

     Error handling
   - WebSocket Disconnections (Implement automatic reconnection with loading dialog).
   - Backend service failures ( display a clear message indicating that real-time updates are temporarily unavailable and fallback to polling ).
   - Push notification failures ( user is disable permission of recive notification or close the notifiction channel so we will need to show dialog to tell user that we need to grantue       the notifiction permission with       
     button go to setting ) 
   - Data inconsistency ( if the response model of notification changed app will crash so we make data validation to make sure it's the needed data).
   - handle cases by make force call for the get orders status api to update the orders status if there is an unexpected error.

     Tech stack reasoning 
   - Platform:Kotlin (Android).
   - WebSocket Libraries: Libraries available for both platforms (socket.io , OkHttp or Java-WebSocket for Android).
   - FCM SDKs: Provided by Google for handling push notification registration and delivery.

3.Mentorship Plan
     How would you onboard a junior engineer to this feature?
   - explaining the goal of the feature — real-time order updates — and the importance of ensuring the system scales and remains reliable under high load.
   - walk them through the core tech stack (socket.io,mvvm pattern,clean arch).
     
     What would you delegate to them?
   - starting with ui implementation and then go for layers like ( domain and data ).
   - focus with him on pr reviews to gain experiance.
   - writing data class that mock real time response.
   - setup every part in his place and start to test the happy path with him.
   - should the out source code be clean, modular, and testable.

     How would you help them understand the architecture and write tests?
   - Best practices for asynchronous programming and error handling in real-time systems.
     
     What pitfalls would you help them avoid?
   - Avoiding common pitfalls, such as over engineering the task or over-complicating state management.
   - Avoiding ignoring connection loss what will do if the connection is down ?
   - Ensure that the app doesn’t update too many state changes in a short time.
     
4. Optional (Bonus)
   Fallback strategy if real-time connection fails
   - If WebSockets fail, use HTTP polling as a fallback for order status updates.
   







