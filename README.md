#### Pydantic Validation and Parsing code along 
Within the server directory create a file called validate.py where we shall write code for this code along

In this code along we will demonstrate validation with a user profile model with the fields: first_name, last_name, email, password and age.   

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

    
Before we add validation to the fields above we will start by making deliberate mistakes to see how pydantic behaves by default by providing wrong types and not setting some fields.   
#### Excluding a field 
As shown below create an instance of the model above and intentionally exclude the email field. 
```    
u1 = User(first_name="Ada", last_name="Lovelace", password="adalve123", age=34)
print(u1)
```
When we run the code `python3 validate.py` we will get a validation error for the email.    

![Email Validation Pydantic](./email%20validation%20error%20Pydatic.png)     

To avoid getting this error let us add a default value to the field.    

`email : str ='email@domain.com' `      

When we run our code again we do not get an error and the email field is automatically populated to the default we set.   

If you were apprehensive about us adding a default value to the email field then you deserve a pat on the back as your software engineering instincts are beyond active. Pardon me as we are only demonstrating a few default characteristics of pydantic. A more effective way would be to make all the values None by default, making it easy to identify the unset values and supply the necessary logic dealing with the unset values.  

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

```
from typing import Optional      

class User(BaseModel):
    first_name: Optional[str] = None
    last_name: Optional[str] = None 
    email: Optional[str] = None
    password: Optional[str] = None
    age: Optional[int] = None
```   
The code above gives the unset values a default value of None when they are not set.    
Let us demonstrate with an example.     
```
    user1 = User()
    print(user1)
```     
The results will be as shown below. 

![Unset Values Example](./None_Values_Result.png)   


### Add invalid types  
The next intentional mistake we want to make is add invalid data types to some of our fields.    
Adding a number string to the age field that requires an int.   

` age='34' `  

```
user2 = User(first_name="Ada", 
            last_name="Lovelace",
            email="adalve@gmail.com",
            password="adalve123",
            age='34') 
print(user2)
```     
Lucky enough when we run our code we do not get an error. This is because the value we provide is automatically coerced into an int. This is never always the case as it will depend on wether the value can be coerced into the required type.  

The code below would throw an error since the string 'f' cannot be coerced into an int.      
```
user3 = User(first_name="Ada", 
            last_name="Lovelace",   
            email="adalve@gmail.com",   
            password="adalve123",   
            age='f34')
print(user3)
```     
![Error for incompatible type](./Error%20for%20invalid%20input-Pydantic.png)  

A similar type exception would be thrown if an str field receives an int. You can further explore with different incompatible types to be more informed on how coercion works by default in pydantic.        

Let us now add validation to our fields according to the criteria set above. We will combine the constrain field functions and the filed_validator decorator function to enforce validation.  

```
import re

from pydantic import BaseModel,constr,conint,field_validator,ValidationError

class User(BaseModel):
   
    first_name: constr(min_length=2,max_length=20) # length constrains added
    last_name: constr(min_length=2,max_length=20) # length constrains added
    email: constr(min_length=7,max_length=50) # length constrains added
    password: constr(min_length=9,max_length=50) # length constrains added
    age: conint(strict=True,gt=14,le=70) # range constrains added

    @field_validator('first_name','last_name')
    @classmethod
    def validate_name(cls, value, info:FieldValidationInfo):
        #check if the value supplied supplied for first_name and last_name contains a number
        if re.findall(r'\d',value):
            raise ValidationError('Name must not contain a number')
        #check if the value supplied for first_name and last_name contains a special character
        if re.findall(r'\W',value):
            raise ValidationError('Name must not contain special character')
        return value.title()
    
    @field_validator('password')
    @classmethod
    def validate_password(cls, value):
        #check if the password contains a number
        if not re.findall(r'\d',value):
            raise ValidationError('Password must contain a number')
        #check if the password contains a special character
        if not re.findall(r'\W',value):
            raise ValidationError('Password must contain a special character')
        #check if the password contains a Capital Letter
        if not re.findall(r'[A-Z]',value):
            raise ValidationError('Password must contain a capital letter')
        #check if the password contains a lowercase character
        if not re.findall(r'[a-z]',value):
            raise ValidationError('Password must contain a lowercase letter')
        return value
  
    @field_validator('email')
    @classmethod
    def validate_email(cls, value):
      mailregex = re.compile(r"([-!#-'*+/-9=?A-Z^-~]+(\.[-!#-'*+/-9=?A-Z^-~]+)*|\"([]!#-[^-~ \t]|(\\[\t -~]))+\")@([-!#-'*+/-9=?A-Z^-~]+(\.[-!#-'*+/-9=?A-Z^-~]+)*|\[[\t -Z^-~]*])")
      if re.fullmatch(mailregex,value):
         return value.lower()
      raise ValidationError("Error in the email entered")


    
user6 = User(first_name="John",    
        last_name="Doe",    
        email="jdoe@gmail.com", 
        password="Doer5%a23ke^",    
        age=70, 
        date=date(2023,9,2))
print(user6)

```  
In implementing the validation above we have incorporated 2 methods.    

We have used constraining fields to constrain length of the fields.   

```
first_name: constr(min_length=2,max_length=20) # length constrain added for first_name field
...
password: constr(min_length=9,max_length=50) # length constrain added for password field
```    

We have further used the field_validator decorator to enforce validation on the fields.     

```
    @field_validator('password')
    @classmethod
    def validate_password(cls, value):
        # check if a number is present in the value provided 
        if not re.findall(r'\d',value):
            raise ValidationError('Password must contain a number')
        # check if a special character is present in the value provided 
        if not re.findall(r'\W',value):
            raise ValidationError('Password must contain a special character')
        # check if a Capital letter is present in the value provided 
        if not re.findall(r'[A-Z]',value):
            raise ValidationError('Password must contain a capital letter')
        # check if a lowercase character is present in the value provided 
        if not re.findall(r'[a-z]',value):
            raise ValidationError('Password must contain a lowercase letter')
        # if all checks above pass then return the value
        return value

```        
This brings us to the end of the pydantic code along.

Next we shall have a look how to make patch and delete requests in fastapi.  
#### sources         
[Pydantic Version 2.2](https://docs.pydantic.dev/latest/)      
[Validity - What-are-the-rules-for-email-address-syntax](https://knowledge.validity.com/hc/en-us/articles/220560587-What-are-the-rules-for-email-address-syntax-)   
[Stackabuse Email Regex For Validation](https://stackabuse.com/python-validate-email-address-with-regular-expressions-regex/)    

