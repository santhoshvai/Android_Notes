## Example
![Imgur](http://i.imgur.com/9rl6PBX.png)

## Contract
* It is an agreement between data-model, storage, and views, presentation, **describing how information is accessed.**
* Implemented in a contract class that contains a list of constants, typically database column names, that are used to asscoiate data from the data source within the UI of an application.

![Imgur](http://i.imgur.com/jVQwoI6.png?1)

* Each column in the data source represents an UI element
* [Example - contactscontract](http://developer.android.com/intl/zh-cn/reference/android/provider/ContactsContract.html)
  * It has a cloumn for the display name
  * Flags for the contact
  * columns for IDs that are used to link the contact to data in other tables
