# Two Factor Authentication Tutorial

The tutorial starter files are found in the StarterCode directory at https://github.com/ConnorWatsonGitHub/2FactorAuthenticationDemo.  

Completing the tutorial will result in the code found in the FinalDemo directory, implementing 2FA functionality. 

## Part 1:  Set Up Initial Project with Basic Login Strategy

### Clone repo and open StarterCode project 

The cloned starter code provides a base Express project, with handlebars templating and basic login function.
Basic login sets up the first authentication factor. 

*Note:  If you prefer to create the project from scratch, find a basic overview for creating the base project at the end of this ReadMe.*

## Part 2:  Implementing 2FA – TOTL method with Google Authenticator and QR Codes
TOTL (Time-Based One-time Password) first requires the user to be pre-authenticated with basic login credentials.

Then the user is given a QR code, which they scan using Google Authenticator app.
Google Authenticator provides user with an authentication token, which is entered into the site.
When the resulting token is verified, 2FA is completed and the user is fully authenticated.

*Important: To test this method, you will need to download the Google Authenticator App for your phone.*

### Prepare Project File Structure 
- In routes folder, add file googleAuthenticator.js
- In views folder, add file account.hbs
- In views folder, add a new folder googleAuthenticator
- In views/googleAuthenticator folder, add 3 files: index.hbs, twoFactorSuccess.hbs, validateCode.hbs

### Set up project to store the user's 2FA method preference
- In models/user.js, add 2 fields to schema definition:  
  `secretKey: String,
   twoFAMethod: String`
- In views/account.hbs, create button which will direct the user to the 2FA view

  `<a class="btn btn-primary" href="./googleAuthenticator/" >Google Authentication</a>`

### Connect authenticator router in app.js
- Above the index router require, insert `var googleAuthenticatorRouter = require('./routes/googleAuthenticator');`
- Above the index router use, insert `app.use('/googleAuthenticator', googleAuthenticatorRouter);`

### Install the Required NPM Packages
- In terminal, be sure you are in the project directory, then run these commands:
  - `npm i speakeasy`
  - `npm i node-qrcode`

### Configure the authenticator router
- Open routes/googleAuthenticator.js
- Import express, passport and the user model, and create the router object
  
  `var express = require('express');
  var router = express.Router();
  const User = require('../models/user');
  const passport = require('passport');`
- Import the npm modules

  `const speakeasy = require("speakeasy");   
   const QRCode = require('qrcode');`
- Add the IsLoggedIn middleware function for checking that the user is logged in and authenticated

  `function IsLoggedIn(req, res, next) {
    if (req.isAuthenticated()) {
        return next();
    }
    res.redirect('/login');
}`
- Configure GET Handler for /
  - This generates a secret key via speakeasy, stores it in the user's file, and then shows QR code
 
    `router.get('/', IsLoggedIn, function (req, res, next) {
        var encodedKey = speakeasy.generateSecret().base32;
        var otpUrl = 'otpauth://totp/' + req.user.username +
             '?secret=' + encodedKey + '&period=30';
        var qrImage = 'https://chart.googleapis.com/chart?chs=166x166&chld=L|0&cht=qr&chl=' + encodeURIComponent(otpUrl);        
        User.findOneAndUpdate({
                _id: req.user._id
            }, {
                secretKey: encodedKey,
                twoFAMethod: "google"
            },
            function (error, success) {
                if (error) {
                    console.log(error);
                } else {
                    console.log(success);
                }
            });        
        res.render('googleAuthenticator/index', {
            user: req.user,
            qrImage: qrImage,
            key: encodedKey,
        });    
});`

 - Configure GET Handler for /validateCode
  
   `router.get('/validateCode', IsLoggedIn, (req, res, next) => {
    res.render('googleAuthenticator/validateCode', {
        title: 'Add Recommendations',
        user: req.user
    });
});`

  - Configure POST Handler for /validateCode
    - This verifies the token using the user's secret key
    
      `router.post('/validateCode', (req, res, next) => {
    let userToken = req.body.code;
    let base32secret = req.user.secretKey;
    let verified = speakeasy.totp.verify({  secret: base32secret,
                                            encoding: 'base32',
                                            token: userToken});
    if (verified) {
        res.redirect('/googleAuthenticator/twoFactorSuccess');
    }
    else {
        res.redirect('/googleAuthenticator/validateCode');
    }
});`

  - Configure GET Handler for /twoFactorSuccess
    - This is the view user sees when 2FA is complete
    
      `router.get('/twoFactorSuccess', IsLoggedIn, (req, res, next) => {
    res.render('googleAuthenticator/twoFactorSuccess', {
        title: 'Two Factor Success',
        user: req.user
    });
});`

  - Finally, at the end of the file, export the router object
  
    `module.exports = router;`

### Prepare the Authenticator Views
- views/googleAuthenticator/index.hbs
  - This view shows to the generated QR code to the user to scan
  - Then asks the user to enter the validation code received from Google Authenticator after scanning the QR code
  
    `{{#if messages}}`
    `<div class="alert alert-danger">`
    `{{messages}}`
    `</div>`
    `{{/if}}`
    `<h3> Scan this QR Code in Google Authenticator </h3> `
    `<img src = "{{ qrImage }}" />`
    `<h3>Please enter code from your Google Authenticator App</h3>`
    `<form method="post">`
    `<div class="enterCode">`
        `<label>Code:</label>`
       ` <input type="text" name="code" id="code" /><br />`
       ` <input name="key" id="key" type="hidden" value="{{key}}" />`
    `</div>`
   ` <div>`
       ` <button>Submit</button>`
    `</div>`
    `</form>`

- views/googleAuthenticator/validateCode
  - This view asks the user to enter the validation code received from Google Authenticator after scanning the QR code
  
    `<form method="post">`
    `<div>`
       `<label>Code:</label>`
        `<input type="text" name="code" id="code" /><br />`
    `</div>`
    `<div>`
        `<button>Submit</button>`
    `</div>`
`</form>` 

- views/googleAuthenticator/
  - This view tells the user they were successfully authenticated with 2FA
  
    `<h1>2 FACTOR AUTHENTICATION WORKED!!!!!</h1>`  
    
### Congratulations, you have successfully implemented 2FA using Google Authenticator!



# Start Here If Creating the Project from Scratch (without cloning)

### Create a new Express project, with handlebars templating:
- Open a new project, open the terminal, navigate to the project directory, then run these commands:
- `npm i express`
- `npm i express-generator`
- `npx express-generator --view=hbs`, then `y` to continue
- `npm install`
- `npm i mongoose`
- `npm i passport passport-local passport-local-mongoose express-session`

### Prepare file structure
- In root folder, add config directory, containing file globals.js
- In root folder, add models directory, containing file user.js
- In routes folder, delete file users.js
- In views folder, add 3 files: login.hbs, loginSuccess.hbs, register.hbs

### Modify app.js
- Delete line 8: `var usersRouter = require('./routes/users');`
- Delete line 22: `app.use('/users', usersRouter);`
- Import the required modules and models 
- Configure MongoDB Connection (secure your connection string by putting it in config/globals.js)
- Initialize Passport and configure Passport Session Cookie
- Create Passport Strategy and Serialization for User  

### Add User Model
- In models/user.js, import mongoose and passport-local-mongoose
- Create a new mongoose schema definition for user containing username, password, oauthID, oauthProvider, and created
- Plugin passport-local-mongoose and export the schema as a new User model

### Add Views
- In views/login.hbs, create a form (post method) to login user, with inputs for username and password and a login button.
- In views/register.hbs, create a form (post method) to register user, with inputs for username, password, confirm password, and register button
- Add any desired custom design/styling

### Add Routes
- In routes/index.js
  - add middleware function IsLoggedIn for checking if user is logged in and authenticated via passport
  - add the following routes:
   - get and post handlers for /login
   - get and post handlers for /register
   - get handler for /loginSuccess
   - get handler for /logout

### Proceed to Part 2 of Tutorial




