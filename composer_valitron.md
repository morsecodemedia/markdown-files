**NOTE: This is still a working document and has yet to be claimed as D&T dogmatic law.**

# Assumptions
This documentation is written with the assumption that you are using: 

 - Composer is installed and being used for dependency management
   - vlucas/valitron - Valitron is a simple, elegant, stand-alone validation library with NO dependencies (https://github.com/vlucas/valitron)

## Need To Look Into
- Composer
   - codeguy/upload - File uploads with validation and storage strategies  (https://github.com/codeguy/Upload)
      - Didn't test yet.

# Valitron
## Installing Valitron
Based on the assumptions listed above, I am not going to go into how to install/use Composer. If you need help with that you can go [here](https://getcomposer.org/) and read up on it.   


In your project folder, install Valitron.

`php composer.phar require vlucas/valitron`  

## Post Array
For the sake of this example, I am going to just use an array I manually created.

    $testArray = array("first_name" => "",  
    "last_name" => "Morse",  
    "email_address_1" => "bmorse@allwebcafe.com",  
    "email_address_2" => "bmorse@wag.agency",  
    "email_address_confirm" => "", //honey pot  
    "street_address_1" => "201 Jay St.",  
    "street_address_2" => "TH E63",  
    "city" => "Stowe",  
    "state_1" => "PA",  
    "state_2" => "Pennsylvania",  
    "postal_code_1" => "19464",  
    "postal_code_2" => 19464,  
    "postal_code_3" => "08330-3414",  
    "country_1" => "United States of America",  
    "country_2" => "US",  
    "phone_1" => "(609) 457 5994 df",  
    "phone_2" => "(609) 457 5994",  
    "phone_3" => "+1 609 457 5994",  
    "phone_4" => "609.457.5994",  
    "phone_5" => "+16094575994",  
    "phone_6" => "609-457-5994",  
    "phone_7" => "457 5994",  
    "phone_8" => "+39055218442",  
    "phone_9" => "+39 055 218442",  
    "website_1" => "http://www.morsecodemedia.com/",  
    "website_2" => "https://www.morsecodemedia.com/",  
    "website_3" => "http://morsecodemedia.com/",  
    "website_4" => "http://morsecodemedia.com",  
    "website_5" => "www.morsecodemedia.com",  
    "website_6" => "morsecodemedia.com/",  
    "website_7" => "morsecodemedia.com",  
    "website_8" => "http://www.wag.agency",  
    "website_9" => "wag.agency"  
    );

## Validating Post Array
To simplify things, we are create an instance of the class, pass it the post array and assign this to a variable.  

`$v = new Valitron\Validator($testArray);`  

Now we set the validation rules. I am going to just do a few, you can get the full list [here](https://github.com/vlucas/valitron#built-in-validation-rules).  

    $v->rule('required', array('first_name','last_name','email_address_1'));  
    $v->rule('email', array('email_address_1','email_address_2'));  
    $v->rule('alpha', array('first_name','last_name'));  
    $v->rule('numeric', array('postal_code_1','postal_code_2','postal_code_3'));  
    $v->rule('url', array('website_1','website_2','website_3','website_4','website_5','website_6','website_7','website_8','website_9'));  
    $v->rule('urlActive', 'website_9');    
  

Now we validate the post.  

    if ( $v->validate() ) {
      echo "<pre> ".__FILE__.": ".__FUNCTION__.": ".__LINE__." \n"; print_r("Validation is all good"); echo "</pre>";
    } else {
      echo "<pre> ".__FILE__.": ".__FUNCTION__.": ".__LINE__." \n"; print_r($v->errors()); echo "</pre>";
    }

### Error Messages
I specifically made sure that there were some values that didn't validate so we can see the error messages. As you can see above, I am calling the error report with `$v->errors()`  
So let's see what we get in return: 

    Array
    (
        [first_name] => Array
            (
                [0] => First Name is required
                [1] => First Name must contain only letters a-z
            )
    
        [postal_code_3] => Array
            (
                [0] => Postal Code 3 must be numeric
            )
    
        [website_5] => Array
            (
                [0] => Website 5 not a URL
            )
    
        [website_6] => Array
            (
                [0] => Website 6 not a URL
            )
    
        [website_7] => Array
            (
                [0] => Website 7 not a URL
            )
    
        [website_9] => Array
            (
                [0] => Website 9 not a URL
                [1] => Website 9 must be an active domain
            )
    
    )

As you can see, it returns a multi-dimensional array. We are given the keys of the post that has an error and then an array with the error message(s) that applies to that invalid input.
				 																																																																																																																					