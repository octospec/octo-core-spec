## Error Reporting:
### Background
1. The OCTo specification distinguishes between retryable and non-retryable errors. Typically any error is non-retryable, but there are a few exceptions.
	* A **non-retryable** error is an error that indicates human intervention is needed. For example:
		* Invalid product / unit ID provided - _This suggests there is a mapping problem due to a changed configuration_
		* Insufficient credits - _The outstanding rolling deposit with the supplier has depleted and a top-up is needed to continue sales. Note that support for this is not included in the core API specification._
		* Authentication failure - _Credentials may need to be updated_
	* A **retryable** error is an error that the consumer of the API generally won't have to take action on. The supplier is likely to find out about the issue and is bound to fix it soon and the call can simply be retried later. This includes situations like:
		* Time outs / down-time
		* Server errors like HTTP 500 without error object in the response body
		* No more availability for a date / timeslot while placing an order
2. Regardless of what type of HTTP status code is returned, an error messages MUST be present.
3. Any response with an HTTP status code outside the 2xx range indicates something is wrong and the response MUST contain an error object.
Typically an HTTP 400 indicates there's something wrong with the input.

The client is expected to detect and handle errors appropriately.

### Error Object
The error response is an array of objects and  MUST contain one or more error objects.
Although interrupting code execution when encountering an error is a valid practice, it is preferred that a list containing all errors (if applicable) is returned as it enables the consumer to appropriately handle each error in independently, such as raising an alarm.

**Single Error Object Structure:**
* `errorCode` contains a namespaced error code. When no specific error code applies, use 1000. In serious cases us 9999 accompanied by an HTTP 500 status code. The namespace is used to distinguish between core and specific capabilities the server may use and must match this regular expression: `/[a-z\-]*\/[0-9]{4}/`.
* `errorMessage` contains a clear, human readable error message that properly describes the issue.
* `fieldName` contains a string identifying the field in the HTTP body or parameter in the URL that the error applies to. Use an empty string if not applicable.


```javascript
[{
	"errorCode": "core/1001",
	"errorMessage": "The `supplierId` provided is missing or invalid.",
	"fieldName": "supplierId"
},
{
	"errorCode": "core/1002",
	"errorMessage": "The `productId` provided is missing or invalid.",
	"fieldName": null
},
{
	"errorCode": "core/1000",
	"errorMessage": "A generic error has occured. Please review your request before trying again.",
	"fieldName": null
}]
```


## Error Codes:
### HTTP level (Global):
|HTTP Status|Retryable|Explanation|
|--|--|--|
|400|Non-Retryable|Invalid request such as missing required query parameters, invalid request model, missing `Timestamp` header that is used to generate HMAC key, etc.|
|401|Non-Retryable|Missing `Authentication` header with HMAC key or key could not be validated|
|403|Non-Retryable|The `Authentication` header was validated but requester does not have correct permissions|
|404|Non-Retryable|Invalid URL path|
|500|Retryable|The server has encountered an error or is otherwise incapable of performing the request. The server _should_ explain the error properly. In exceptional cases where it's unclear what went wrong and the server can't respond in a sensible way then there may not even be an error object in the response body.|
|502|Retryable|An upstream server encountered an error preventing the request from being fulfilled. |
|503|Retryable|The server cannot handle the request (because it is overloaded or down for maintenance). |


### Application Level
Application level errors have codes in different ranges. 
* 1000 - 1499 range for field validation / input errors
* 1501 - 1999 range for business logic violations
* 9999 is for unhandled and unexpected situations
* Capabilities may use overlapping error codes but will be uniquely namespaced. See the capability's documentation for more details.

## Complete Error Overview
### HTTP 400
The errors outlined below are believed to be errors that one will want to act / alert on as they impact the ability to continue the logical call-flow. 
They MUST be accompanied by an HTTP 400 status code

**Generic**
- `ErrorCode` 1499: Something is wrong with the input. `errorMessage` should indicate what exactly.
- `ErrorCode` 1498: Unable to create entity. `errorMessage` should indicate what exactly. Use when unable to process a POST request and no more explicit error code is available.
- `ErrorCode` 1497: Unable to update entity. `errorMessage` should indicate what exactly. Use when unable to process a PUT/PATCH request and no more explicit error code is available.

**Supplier** 
No errors identified

**Products**
- `ErrorCode` 1001: `supplierId` not found / invalid
- `ErrorCode` 1002: `productId` not found / invalid
- `ErrorCode` 1003: `optionId` not found / invalid

**Availability**
- `ErrorCode` 1001: `supplierId` not found / invalid
- `ErrorCode` 1002: `productId` not found / invalid
- `ErrorCode` 1003: `optionId` not found / invalid
- `ErrorCode` 1004: `availabilityId` not found / invalid

**Reservations**
- `ErrorCode` 1001: `supplierId` not found / invalid
- `ErrorCode` 1002: `productId` not found / invalid
- `ErrorCode` 1003: `optionId` not found / invalid
- `ErrorCode` 1004: `availabilityId` not found / invalid
- `ErrorCode` 1005: `uuid` of reservation/booking not found / invalid
- `ErrorCode` 1501: Reservation expired 
- `ErrorCode` 1502: Hold period too long 
- `ErrorCode` 1503: Cannot extend hold period 
- `ErrorCode` 1504: Reservation ID not found
- `ErrorCode` 1505: No more availability

**Bookings**
- `ErrorCode` 1001: `supplierId` not found / invalid
- `ErrorCode` 1005: `uuid` of reservation/booking not found / invalid
- `ErrorCode` 1501: Reservation expired 

**Cancellations**
- `ErrorCode` 1005: `uuid` of reservation/booking not found / invalid
- `ErrorCode` 1506: Cancellation not allowed

### HTTP 500
System errors that occur in the software that embeds the OCTo specification may cause erratic behavior or cause hard-to-map errors. Those should be thrown with an HTTP 500 status code.
```javascript
[{
	"errorCode": 9999,
	"errorMessage": "System.outOfMemoryException thrown in /some/file",
	"fieldName": null
}]
```
### Other guidelines
- An error during a "create" action such as placing a booking should be a guarantee that the booking was not finalized and shall not be charged. **ONE FOR THE SLA MAYBE?**




## NOT FOR PUBLISHING
### Error handling thoughts and context**
> * There could be many different errors which could be caused by an invalid request. Only the ones that are likely to be non-retryable have been assigned their own error code.
> * Errors that happen in the software that embeds the OCTo specification are not easy to standardize. The standard response for that type of error should be of HTTP 500 as it suggests something more serious went wrong, such as:
>    * Being unable to connect to a database
>    * An error occurring in the booking software and it's not reasonably possible to return even a proper error object can be returned.
> * One is expected to use HTTP 400 often.
> * An exhaustive list of all possible request object fields that could be missing or invalid has been skipped, but _may_ be added in the future.
> * Per definition an error is non-retryable. Only non-retryable errors need to be explained and have an error code so that they can be handled programmatically
### Ground rules for extending this document
* New error codes MAY be introduced for "*non-retryable*" errors only. A retryable error suggests that it is actually a warning (which the API specification doesn't support) or it is an error due to a failure in an upstream system. (So not in the system you are communicating with directly, but another system further down the line that it relies upon.)
* Identical errors situations MUST be assigned a single error code where possible.
