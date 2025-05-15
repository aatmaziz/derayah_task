Overview
This document outlines the design and partial implementation of a Real-Time Order Status and Trade Notifications feature for a cross-platform trading application (Mobile and Web). The goal is to enable users to receive instant updates on their open orders and trade executions without manual refreshes, ensuring a seamless and responsive user experience.

Requirment

"We want users to instantly see updates on their open orders and executions without refreshing manually."
This case study evaluates the ability to:

Interpret and clarify vague requirements.

Architect scalable, clean systems.

Implement critical components.

Mentor junior engineers effectively.

🌆 Scenario
As the Staff Engineer, you are responsible for delivering this cross-platform capability. Users must receive real-time updates as their orders transition through states (e.g., Submitted, Partially Filled, Filled, Cancelled, Rejected) across various asset types (stocks, mutual funds, ETFs, crypto). You have full control over technology choices, leveraging existing backend services and trading execution platforms.


1. Problem Analysis

Assumptions

To address the requirements, the following assumptions are made:

Existing Backend Services: Services for user authentication and login are already in place.

CRUD Operations: APIs exist for creating, modifying, and cancelling orders.

Order Retrieval: A service fetches order data from execution platforms.

Platform Identification: A field identifies whether the user is on Mobile or Web.


Unified Order Model: All integrated platforms transform order data into a consistent internal model.


Order JSON Structure:

{
  "symbol": "string",
  "id": "string",
  "created_at": "timestamp",
  "quantity": "number",
  "price": "number",
  "order_type": "string",
  "platform": "string"
}

Open Questions

To clarify the requirements, the following need resolution:

Update Interval: What is the expected frequency for order status updates?

Database Type: Is it SQL (e.g., PostgreSQL), NoSQL (e.g., MongoDB), or another type?

User Scale: How many concurrent users are expected?

Order Volume: What is the average and peak number of orders processed?

Aggregation Needs: Are order updates aggregated (e.g., batched) or delivered individually in real-time?

Vague Requirements

The requirements lack specificity in the following areas:

"Without Refresh": Implies a real-time connection (e.g., WebSockets), but the specific library or protocol is not defined.

Latency Expectations: No clear definition of acceptable latency for updates (e.g., <1s, <5s).

Notification Content: The exact content and format of notifications (e.g., text, data fields) are unspecified.

MVP Scope vs. Full Solution

MVP Scope

The Minimum Viable Product (MVP) focuses on core functionality:

WebSocket Integration: Enable real-time updates for order statuses (Submitted, Partially Filled, Filled, Cancelled, Rejected).

Basic Error Handling: Display user-friendly error messages for connection issues or failed updates.

UI Implementation: Build an orders list screen with pagination using a paging library (e.g., Jetpack Paging for Android).

Connection Status Indicators: Show "Connection Lost" warnings and implement auto-reconnect logic.

Full Solution

The complete solution extends the MVP with:

Latency Optimization: Minimize update latency (e.g., sub-second delivery).

Fallback Mechanism: Use HTTP polling if WebSocket connections fail.

Background Updates: Support background threads for data updates, ensuring seamless UX.

Precise Timestamps: Include execution timestamps for user-friendly feedback (e.g., "Filled 2 seconds ago").

User Journeys and Edge Cases

User Journey

User logs into the mobile app or web platform.

User navigates to a stock details screen.

User creates and submits an order.

User is redirected to the orders history screen.



Order status updates to "Partially Filled" in real-time.



Order status updates to "Filled" in real-time.

Edge Cases





Connection Loss: Fallback to polling every X seconds or display a retry dialog to fetch order updates.



Multi-Device Login: Handle simultaneous logins on multiple devices, ensuring all receive updates or gracefully handle logout from one device.



Order Rejections/Cancellations: Display clear error messages for rejected or cancelled orders.



2. System Design Document _

Architecture Diagram

Refer to the architecture diagram in the diagram/ directory for a visual representation of the system.

Real-Time Mechanism





WebSockets: Used for live updates when the app is in the foreground, ensuring low-latency delivery.



Firebase Cloud Messaging (FCM): Delivers push notifications for mobile apps in the background, summarizing order updates.

Platform-Specific Data Flow (Mobile)





User Action: The user submits an order via REST APIs.



Backend Processing: Backend services validate and process the request, updating the database.



Event Notification: The Real-Time Notification Service identifies the user's mobile device(s).



Foreground Delivery:





If the app has an active WebSocket connection, updates are sent via WebSocket.



Background Delivery:





If the app is in the background, the Notification Service sends a push notification via FCM with a brief update summary.



App Reactivation:





When the user opens the app (via notification or directly), it either:





Establishes a WebSocket connection for real-time updates.



Fetches the latest order data via REST API calls.

Error Handling





WebSocket Disconnections:





Implement automatic reconnection with a loading dialog to inform users.



Backend Service Failures:





Display a message indicating temporary unavailability of real-time updates and fallback to polling.



Push Notification Failures:





If notifications are disabled, prompt the user with a dialog to enable permissions, including a button to navigate to settings.



Data Inconsistency:





Validate incoming notification data to prevent crashes due to model changes.



Unexpected Errors:





Force-call the "Get Orders Status" API to refresh order statuses if errors occur.

Tech Stack Reasoning





Platform: Kotlin for Android, leveraging its modern features and ecosystem.



WebSocket Libraries:





Options include socket.io for simplicity, OkHttp for robust WebSocket support, or Java-WebSocket for lightweight integration.



FCM SDK: Google’s official SDK for reliable push notification handling.



UI Framework: Jetpack Compose for Android to build a reactive, modern UI.



3. Mentorship Plan

Onboarding a Junior Engineer

To onboard a junior engineer:





Explain the Feature Goal: Highlight the importance of real-time order updates for user experience and the need for scalability under high load.



Walk Through the Tech Stack:





Introduce socket.io for WebSocket communication.



Explain the MVVM pattern and Clean Architecture principles for modular code.



Demonstrate the Android Paging library for efficient data loading.

Tasks to Delegate





UI Implementation: Build the orders list screen using Jetpack Compose and the Paging library.



Data Layer: Implement data classes to mock real-time WebSocket responses.



PR Reviews: Involve them in code reviews to gain experience and understand best practices.



Testing Happy Path: Guide them to set up and test the primary user journey (e.g., order submission to Filled status).



Code Quality: Ensure their code is clean, modular, and testable.

Teaching Architecture and Testing





Architecture:





Use diagrams to explain data flow (UI → ViewModel → Repository → WebSocket/REST).



Discuss Clean Architecture layers (Presentation, Domain, Data) and their responsibilities.



Testing:





Teach unit testing for ViewModels and repositories using JUnit and MockK.



Introduce UI testing with Espresso or Compose Testing for user journeys.



Emphasize testing edge cases like connection loss and invalid data.

Pitfalls to Avoid





Over-Engineering: Guide them to keep solutions simple and focused on the MVP.



Connection Loss Handling: Ensure they account for WebSocket disconnections and implement fallback polling.



State Management: Prevent excessive state updates by batching or debouncing rapid changes.



Ignoring Scalability: Stress the importance of efficient data handling for large order volumes.



4. Optional (Bonus): Fallback Strategy

If the WebSocket connection fails:





HTTP Polling: Implement polling at a configurable interval (e.g., every 5 seconds) to fetch order status updates.



User Feedback: Display a subtle "Polling Mode" indicator to inform users of the fallback state.



Reconnection Logic: Continuously attempt to re-establish the WebSocket connection in the background, reverting to real-time updates once restored.



Feature Flag for Critical Bugs: Implement a feature flag (e.g., via a remote configuration service like Firebase Remote Config) to hide the order history screen if a severe bug is detected, preventing users from accessing potentially broken functionality. Display a user-friendly message (e.g., "Order history is temporarily unavailable") and redirect users to an alternative screen (e.g., home or portfolio).

Monitoring and Observability Plan





Crashlytics Integration: Utilize Firebase Crashlytics for mobile apps to monitor and detect bugs in real-time. Crashlytics will provide detailed crash reports, including:





App Version: Identify the specific app version affected by the bug.



Number of Affected Users: Track the scale of the issue by monitoring the number of users impacted.



Stack Traces and Context: Capture detailed error logs to diagnose root causes efficiently.



Actionable Insights: Use Crashlytics data to prioritize bug fixes and inform decisions about enabling/disabling the feature flag for the order history screen.
