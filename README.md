
To demonstrate the use of the various ways of validation we will use a user profile model with the fields first_name, last_name, email, password and age.   

```
from pydantic import BaseModel

class User(BaseModel):
    first_name: str
    last_name: str
    email: str
    password: str
    age: int

```
Our aim is to add the following validation rules to the different fields.     
- first_name and last_name  
     - must not be empty.   
     - must be at least 2 characters and at most 20 characters.   
     - must not contain numbers or special characters.  
- email     
    - Must contain the 4 most essential parts  
        - recipient name (alphanumeric and special chars allowed, special chars must not be first or last)
        - the @ symbol
        - domain name (alphanumeric characters, maybe contain a hyphen and a period , )
        - th period (.) 
        - Top level domain (letters) 
- Password  
        - have more than 8 characters   
        - contain a capital letter   
        - contains lowercase letters    
        - contain a special character.   
- Age   
        - must be an integer.   
        - must not be less than 14.     
        - must not be greater than 70.   

    
Before we add validation to the fields above we will start by making deliberate mistakes to see how pydantic behaves by default by not setting some fields and providing wrong types to some fields.    
Create an instance of the model above and intentionally exclude the email field. 
```    
u1 = User(first_name="Ada", last_name="Lovelace", password="adalve123", age=34)
print(u1)
```
When we run the code we will get a validation error for the email as below.    

![Email Validation Pydantic](./enail%20validation%20error%20Pydatic.png)    
To avoid getting this error we can add a default value to the field this way.   
`email : str ='email@domain.com' `  
When we run our code again you will notice that now we do not get an error and the email field is automatically populated to the default we have set. Now of course you may be right if you have judged that this is not the best approach to handling emails and probably any other fields. A better approach would be to make the values None by default for most use cases and provide the necessary logic for handling unset values for the functionality you would like to achieve.    
Example:
```
class User(BaseModel):
    first_name: str = None
    last_name: str =None
    email: str = None
    password: str = None
    age: int = None
```
Another more explicit way would be to make use of the Optional attribute from the typing module.  
` from typing import Optional `     
```
class User(BaseModel):
    first_name: Optional[str] = None
    last_name: Optional[str] = None 
    email: Optional[str] = None
    password: Optional[str] = None
    age: Optional[int] = None
```     
The next intentional mistake we want to make is add invalid data types to some of our fields.    
Adding a string to the age field that requires an int.    
```
u1 = User(first_name="Ada", last_name="Lovelace",email="adalve@gmail.com",password="adalve123", age='34')
print(u1)
```     
On running our code we get do not get an error as the value we provide is automatically coerced into an integer. This is never always the case as it will depend on wether the value can be coerced into the required type.     
The code below would throw an error.      
```
u1 = User(first_name="Ada", last_name="Lovelace",email="adalve@gmail.com", password="adalve123", age='f34')
print(u1)
```     
![Error for incompatible type](./Error%20for%20invalid%20input-Pydantic.png)    
The same exception would be thrown if an str field receives an int. You can further  explore with different incompatible types to be more informed on how coercion works by default in pydantic.  