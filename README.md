# FirebaseAuthenticationTutorial
Learn how to implement Firebase Authentication into an Android app


## Project Documentation

**Step 1: Creation** <br/>
I created a new android porject in java and placed it in this repo.

**Step 2: Dependancies** <br/>
Add Firebase Authentication to dependancies in build.gradle (Module).
**Note:** When creating the firebase project there is steps to import the google-services file that you will need in your java folder. See: [Firebase Auth Documentation](!https://firebase.google.com/docs/auth)


Add this line in the file (Just below the plugins section, not in the section):
```java
apply plugin: 'com.google.gms.google-services'
```

Add these lines in the dependancies section:
```java
    implementation 'com.firebaseui:firebase-ui-auth:7.2.0'
    implementation platform('com.google.firebase:firebase-bom:29.0.0')
```
<br/>

Then run a project sync.

<br/>

**Step 3: Design and Implement LogIn and SignUp UIs** <br/>
I created two simple pages. The both of them looking like sisters, they have their title, email field, password field and submit button.
In addition this this I had to add a menu in order to switch between the pages.

**Step 4: Implement SignUp with Email** <br/>
I first created a method that handles the submit button click. This indicates that the user wants to sign up. 
I had to create the variables that represent the UI elements, I then assigned them to the appropriate element.
I then extracted the value out of them
Then I called the createAccount method and passsed in the two values
```java
    public void onSignUpButtonClick(View view) {
        EditText enteredEmail = findViewById(R.id.activity_signup_email_edittext);
        EditText enteredPassword = findViewById(R.id.activity_signup_password_edittext);

        String email = enteredEmail.getText().toString();
        String password = enteredPassword.getText().toString();
        createAccount(email, password);
    }
``` 
<br/>

In the createAccount method I used an instance of FirebaseAuth to call the createUserWithEmailAndPassword method while passing in the email and password values. I then added a addOnCompleteListener to determine whether or not it was successful, if it was then I would call the updateUser method and pass in the user object. If not then I would still call the updateUI but pass in a value of null. I would also write a toast for both of these outcomes.
```java
    private void createAccount(String email, String password) {
        mAuth.createUserWithEmailAndPassword(email, password)
                .addOnCompleteListener(this, task -> {
                    if (task.isSuccessful()) {
                        // Sign in success, update UI with the signed-in user's information
                        Log.d(TAG, "createUserWithEmail:success");
                        FirebaseUser user = mAuth.getCurrentUser();
                        updateUI(user);
                    } else {
                        // If sign in fails, display a message to the user.
                        Log.w(TAG, "createUserWithEmail:failure", task.getException());
                        updateUI(null);
                    }
                });
    }
```
<br/>

I then was able to use the updateUI method to show the user the toasts and if the signup was successful I would then sent the user to the home page with an Intent.
```java
    private void updateUI(FirebaseUser user) {
        if(user != null) {
            Toast.makeText(SignUpActivity.this, "Sign Up Successful.",
                    Toast.LENGTH_SHORT).show();
            // Navigate to home screen
            Intent goToHomeIntent = new Intent(this, MainActivity.class);
            startActivity(goToHomeIntent);
        }
        else {
            // Send toast with error
            Toast.makeText(SignUpActivity.this, "Sign Up failed.",
                    Toast.LENGTH_LONG).show();
        }
    }
```

**Step 5: Implement LogIn with Email** <br/>
The login was faily similar to the signup
Just like the signup page, I created a method that handles the submit button click.
Them I created the variables that represent the UI elements, I then assigned them to the appropriate element.
I then extracted the value out of them
Then I called the signin method and passsed in the two values
```java
    public void onLoginButtonClick(View view) {
        EditText enteredEmail = findViewById(R.id.activity_login_email_edittext);
        EditText enteredPassword = findViewById(R.id.activity_login_password_edittext);

        String email = enteredEmail.getText().toString();
        String password = enteredPassword.getText().toString();
        signIn(email, password);
    }
```
<br/>

Now tell me if this feels familiar but...
In the signIn method I used an instance of FirebaseAuth to call the signInWithEmailAndPassword method while passing in the email and password values. I then added a addOnCompleteListener to determine whether or not it was successful, if it was then I would call the updateUser method and pass in the user object. If not then I would still call the updateUI but pass in a value of null. I would also write a toast for both of these outcomes.
```java
    private void signIn(String email, String password) {
        mAuth.signInWithEmailAndPassword(email, password)
                .addOnCompleteListener(this, task -> {
                    if (task.isSuccessful()) {
                        // Sign in successful
                        Log.d(TAG, "signInWithEmail:success");
                        FirebaseUser user = mAuth.getCurrentUser();
                        updateUI(user);
                    } else {
                        // Sign in fails
                        Log.w(TAG, "signInWithEmail:failure", task.getException());
                        updateUI(null);
                    }
                });
    }
```
<br/>

I then was able to use the updateUI method again to show the user the toasts and if the signIn was successful I would then sent the user to the home page with an Intent.
```java
    private void updateUI(FirebaseUser user) {
        if (user != null) {
            Toast.makeText(LoginActivity.this, "Login Successful.",
                    Toast.LENGTH_SHORT).show();
            // Navigate to home screen
            Intent intent = new Intent(this, MainActivity.class);
            startActivity(intent);
        }
        else {
            Toast.makeText(LoginActivity.this, "Login failed. Email or Password Incorrect.",
                    Toast.LENGTH_LONG).show();
        }
    }
```

**Step 6: Implement Menu System** <br/>
In order for the user to be able to switch between the login and sign up pages, I created a menu.
I used the same menu to allow the user to sign out from the home page.

![Menu Screenshot](/MarkdownAssets/Menu_screenshot.png)

<br/>

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@+id/menu_activity_main_signout"
        android:title="@string/menu_signout_title" />
</menu>
```

I then edited the appropriate activities to implement it.
```java
    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.signout_menu, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(@NonNull MenuItem item) {
        switch (item.getItemId()) {
            case R.id.menu_activity_main_signout:
            {
                mAuth.signOut();
                Intent goToLoginIntent = new Intent(this, LoginActivity.class);
                startActivity(goToLoginIntent);
                break;
            }
        }
        return true;
    }
```
<br/>

**Step 7: Add onStart Method** <br/>
So I can safely assume that the user doesn't want to login everytime. I needed to make a way to see if the user is logged in when the app started (either as a fresh launch or from memory). 
To do this I created this method in the MainActivity (Home activity):
```java
    @Override
    public void onStart() {
        super.onStart();
        // Check if user is signed in then reload with the appropriate screen
        FirebaseUser currentUser = mAuth.getCurrentUser();
        if(currentUser != null){
            currentUser.reload();
        }
        else {
            // Navigate to Login Activity
            Intent intent = new Intent(this, LoginActivity.class);
            startActivity(intent);
        }
    }
```
This method basically just uses an instance of the Firebase Auth to get the current user. If it doesn then that means the user is logged in and they will be sent to the home screen. If the user is not logged in then they will be sent to the login page.

<br/>
