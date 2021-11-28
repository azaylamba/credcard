# credcard
Basic CRED card management

You have to design & build the backend for the application and the ability for the users to add cards, add transactions, get statements etc.
Your task is to go through below templates/stories and create the app.

The tech stack required is Java and Spring Boot. Preferred IDE is Intellij/Eclipse to import the project directly. Postman tool to test the REST APIs.
Make sure maven and .m2 etc are configured on local.
Preferred machine to run the project is the Windows machine because it might be possible that Mac machines (specially M1) do not support some of the jars used.

How to start:
To start the development a template repo is provided at the Github link –
https://github.com/azaylamba/credcard 
The candidate should make sure Java, Postman & Intellij are already installed on their system before starting the development. They should fork the above repo and clone to their local system. Once it is cloned, they should import the project in intellij and start development.
To check the hosted app in the browser, they can search for http://localhost:8000/ in their browser or postman app after running the main class in IDE.

For persistence of data, H2 database should be used which is in-memory database and dependency for the same is already added in POM. If you are comfortable with any other in-memory database like SQLite or some other, feel free to add the dependency and use that.


H2-Database
Database is automatically configured with a sample table BILLIONAIRES. H2 Console can be accessed with below credentials.
username=sa
password=password
Console link - http://localhost:8080/h2-console
Default JDBC URL: jdbc:h2:mem:testdb
How to connect to JDBC: http://www.h2database.com/html/tutorial.html#connecting_using_jdbc 

Problem Breakdown
Template 1 (Manage Cards)
In this template we will be implementing the functionality to manage cards. Create a table called CARD having the following columns - card_id, user_id, card_name, card_provider, card_type, card_number, expiry_date, date_added, last_update_date, status.
Card is the primary key and will be auto generated, card type can be one of the following - mastercard, visa, rupay. Create an enum for car types and use that.
Story 1 (Add a card)
The task is to create an API “card/add” which will be used for adding different cards on the platform.
The API should take a Card object as input.
Validate that all the required information is present on the object except the card_id,date_added, status_cd and throw error if validation fails.
Date_added, last_update_date should be the current date and status should be active.
Create an entry in the CARD table.
Return appropriate response code from the API in case of successful/unsuccessful operation.
Story 2 (Secure a card)
We would be generating a unique token for the card which can be used for online payments instead of using the card numbers. This unique token would also require a PIN to make payments. The PIN can be set by the user. The unique token should be unique across users and should be valid upto 7 days from the date of generation. E.g. if the token was generated on the 1st of the month, it will not be valid on the 8th.
Create a table called CARD_TOKEN having the following columns - token_code, card_id, creation_date, valid_upto, status. The token_code should be an alphanumeric code of length 10.
The task is to create an API “card/token/generate” to generate a token for the card.
The API should take a card_id and user_id as input.
Validate that the card_id exists and belongs to the input user_id. Throw an error if validation fails.
There should be only 1 active token for a card at any given time, so mark all the existing active tokens of the card to inactive.
Generate a unique alphanumeric code of length 10. You can use any third party library to generate the code. Set this code as token_code for the card.
The creation_date should be the current date and the valid_upto date should be creation_date+6 meaning 7 days.
The status should be “active”.
Create an entry in the CARD_TOKEN table.
Return appropriate response code from the API in case of successful/unsuccessful operation.
Story 3 (Set a virtual PIN)
Create another table called CARD_PIN having the columns - card_id, pin, creation_date, status.
The task is to create an API called “card/pin/set” so that users can set a virtual pin to use along with the token_code. The PIN should be a 4 digit number.
The API should take card_id, user_id and pin as input.
Validate that card_id belongs to the user_id passed.
Existing pin cannot be used for the same card again, so validate that the pin was not used earlier for the card.
There could be only one active pin for a card, so mark the existing active pin to “inactive”.
The creation_date should be the current date and the status code should be active.
Create an entry in the CARD_PIN table.
Return appropriate response code from the API in case of successful/unsuccessful operation.
Story 4 (Get token and pin for a card)
The task is to create an API “card/token/details” to get the virtual token and pin for the card so that the users can use them for online payment.
The API should take a card_id and user_id as input. 
Validate that the user_id belongs to the card_id and throw an error if the validation fails.
Return the token_code and pin for the card_id. Only the active token_code and pin should be returned.
If the valid_upto date of the token is in the past, throw an error saying that there is no active token for the card.
Return appropriate response code from the API in case of successful/unsuccessful operation..
Template 2 (Transaction Management)
In this template we are going to develop the functionality to keep track of transactions related to cards. Create a table called TRANSACTION having the following columns - transaction_id, card_id, transaction_amount, transaction_date, transaction_type. Card_id should have foreign key constraint with the CARD table.
Story 1 (Add a transaction)
The task is to create an API “transaction/add” so that a transaction can be added.
The API should take a Transaction object as input.
Transaction date should be the current date and transaction_type can debit or credit. Use an enum for transaction type.
Insert a record in the TRANSACTION table.
Return an appropriate response code from the API.
Story 2 (Generate statement for the card)
Create an API called “card/statement/generate” to generate a statement for the card.
The API should take card_id and a date range as input.
Get all the transactions for the card in the given date range.
Return a list of Transaction objects from the API.
Return an appropriate response code from the API.


Template 3 (Billing)
Story 1 (Generate bill)
Create a table called BILL having the following columns - bill_id, card_id, bill_month, bill_amount, creation_date, due_date, bill_payment_date, status.
The task is to create an API called “card/bill/generate” to generate the monthly bill for the card. This API would be called automatically at the 1st of the month to generate the bill for the previous month.
Since we will be generating the bill for transactions, we need to mark the transactions as billed. So add a column ‘billed’ in the TRANSACTION table so that we can set it’s value as Y upon bill generation.
The API should take card_id as input. Since we are assuming that the API would be called automatically, we are not providing month as input.
Get all the transactions in the previous month and calculate the bill amount for the card. E.g. if there are 3 transactions - debit 10, credit 5, debit 20 then bill amount would be 25.
The current date could be 1st of the month or sometimes second of the month, so we cannot assume that it is always 1st. This is important because we want to calculate the bill for the previous month. You need to check the current month and then generate a bill for the previous month.
The creation_date should be the current_date and the due_date should be the 15th of the current month.
The bill_payment_date should be null and the status should be “due”.
Finally set the value of column ‘billed’ to Y for all these transactions to mark them as billed.
Return appropriate response code from the API in case of successful/unsuccessful operation.
Story 2 (Get unbilled transactions)
The task is to create an API “transaction/unbilled” to get the unbilled transactions for a card.
The API should take card_id as input.
Check for all the transactions where the value of ‘billed’ is not Y in the TRANSACTION table and create a list.
Return a list of Transaction objects from the API. If there are no unbilled transactions, return an empty list.
Return appropriate response code from the API in case of successful/unsuccessful operation.
Story 3 (Pay bill)
The task is to create an API “card/bill/pay” to pay bills for a card. Here we will not be dealing with the actual payment, so we will be just adding the entries in the tables to mimic the payment of the bill.
The API should take card_id as the input.
Check for the bill_amount for the due bills for the input card.
Add a credit transaction in the TRANSACTION table for the card. The transaction_amount should be equal to the bill_amount for the due bill.
Update bill_payment_date to current date and the status to ‘paid’ for the bill_id.
It could be possible that there are multiple due bills for a given card, so handle those scenarios as well. You need to make payment for all the due bills.
Return appropriate response code from the API in case of successful/unsuccessful operation.
