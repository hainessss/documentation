# HS Notifications Service

The HS Notifications Service provides an API for publishing notifications that will resolve to a Platform User and propgate to their UI across the app suite.

The API is exposed at following locations: https://noise.hotschedules.io/, https://noise.bodhi-dev.io/, https://noise.bodhi-qa.io/.

# Types

## ClarifiNotification Type

#### Description

A type used to store notifications. Each notification resolves to one user. Notifications can not be deleted via the api; they can only be dismissed. Notifications will automatically be deleted from the service after 14 days.

#### Properties

 - **recipient_id** - (String) *required* - Username to which the notification is relevant.
 - **notification_type** - (Enum["insight", "alert", "information", "action"]) *required* -  Type of notification. Controls the how the notification is represented in the UI.
 - **unread** - (Boolean) - Wether the notification has been read or not.
 - **dismissed** - (Boolean) - Wether the notification is relevant anymore. Use in place of Deleting notification instances.
 - **title** - (String) - Title of notification.
 - **description** - (String) *required* - The content of the notification.
 - **callout** - (String) - Text used to provide additional context about the notification. Represented in the UI as block-quote.
 - **originator** - (Array(Enum['reports', 'forecasting', 'workflow', 'messages','store_logs','inventory','labor','cash'])) *required* - The Clarifi Module that originated the notification. If you don't know which category your app falls under. Ask your local Product Manager.
 - **action** - (Object) - An object used to direct the user to wherever there is an actionable response to the notification.  
	 - **action.label** - (String) - The label for the button corresponding to the action. 
	 - **action.url** - (String) - The location to where the user is being sent to respond to the notification.

#### System Properties

 - **namespace** - (String) - The namespace to which the notification belongs. (Set using the url parameter). 
 - **sys_created_at** - (Date) - The time when the notification is created.
 - **sys_updated_at** - (Date) - The time when the notification was last updated. 
 - **sys_id** - (ObjectId) - The unique ID.


#### Indexes

- **sys_id** - unique.
- **namespace, recipient_id**
- **dismissed**
- **unread**
- **originator**

## ClarifiNotificationSubscription Type

#### Description

This type can be used to opt out of having notifications being pushed to a particular client. 

#### Properties

 - **username** - (String) *required* - Username to which the subscription is relevant. (The user you want to stop sending notifications to)
 - **unsubscribed** - (Array(Enum['reports', 'forecasting', 'workflow', 'messages','store_logs','inventory','labor','cash'])) - A list of sources from which you do not want to recieve notifications. 

#### System Properties

 - **namespace** - (String) - The namespace to which the subscription is relevant. Set using the url parameter. 
 - **sys_created_at** - (Date) - The time when the subscription is created.
 - **sys_updated_at** - (Date) - The time when the subscription was last updated. 
 - **sys_id** - (ObjectId) - The unique ID.


#### Indexes

- **namespace, username** - unique.

## API

### GET /:namespace/resources/ClarifiNotification/:recipient_id

#### Parameters
- **namespace** - The namespace to which query notifications.
- **recipient_id** - The username of the user to return Notifications for.  

#### Optional Parameters
- **page** - Results page number, defaults to 1.
- **limit** - Number of results to display, defaults to 25.
- **where** - Mongo query object.

#### Description
Returns Notifications that pertain to specific user in a specific namespace.

### GET /:namespace/resources/:type

#### Parameters
- **namespace** - The namespace to which query from.
- **type** - The type to query by.  

#### Optional Parameters
- **page** - Results page number, defaults to 1.
- **limit** - Number of results to display, defaults to 25.
- **where** - Mongo query object.
- **sort** - Mongo sort object.

#### Description
General endpoint for querying types.

### POST /:namespace/resources/ClarifiNotification/users/:username

#### Parameters
- **namespace** - The namespace to which the notification will be associated with.
- **username** - The username to which the notification will be associated with.  

#### Body
ClarifiNotification (JSON)

#### Description
Creates a Notification that pertain to a specific user in a specific namespace.

### POST /:namespace/resources/ClarifiNotification/roles/:role_id

#### Parameters
- **namespace** - The namespace to which the notification will be associated with.
- **role_id** - The role id of the users that the notification will be associated with.  

#### Body
ClarifiNotification (JSON)

#### Description
Creates a Notification that pertain to each User of a specified role.

### POST /:namespace/resources/ClarifiNotification/stores/:store_name

#### Parameters
- **namespace** - The namespace to which the notification will be associated with.
- **store_name** - The name of the store that the notification will be propogated to. 

#### Optional Parameters
- **role_id** - Target users with a specific role within the store.  

#### Body
ClarifiNotification (JSON)

#### Description
Creates a Notification that pertain to each User who exist in the specified store in the setup heirarchy.

### POST /:namespace/resources/ClarifiNotification/branches/:branch_name

#### Parameters
- **namespace** - The namespace to which the notification will be associated with.
- **branch_name** - The name of the branch that the notification will be propogated to. 

#### Optional Parameters
- **role_id** - Target users with a specific role within the branch.  

#### Body
ClarifiNotification (JSON)

#### Description
Creates a Notification that pertain to each User who exist in the specified branch in the setup heirarchy.

### POST /:namespace/resources/:type

#### Parameters
- **namespace** - The namespace to which the type will be associated with.
- **type** - The type to post to.  

#### Body
JSON representing a new type instance.

#### Description
General endpoint for creating instances of types.

### PUT /:namespace/resources/ClarifiNotification/:id

#### Parameters
- **namespace** - The namespace that the notification is associated with.
- **id** - The sys_id of the notification.

#### Body
ClarifiNotification (JSON)

#### Description
This is a REPLACEMENT operation on a ClarifiNotification.

### PATCH /:namespace/resources/ClarifiNotification/:id

#### Parameters
- **namespace** - The namespace that the notification is associated with.
- **id** - The sys_id of the notification.

#### Body
Object(Property Subset of ClarifiNotification) (JSON)

#### Description
This is an Update operation on a ClarifiNotification.

### PATCH /:namespace/resources/ClarifiNotification

#### Parameters
- **namespace** - The namespace that the notifications are associated with.

#### Body
Array of Objects(Property Subset of ClarifiNotification) (JSON). Each object must have a sys_id property.  

#### Description
This is a bulk Update operation on a ClarifiNotification.  







