---
title: MemberPress Forms Customization
teaser: The struggle is real.....
category: tech
tags: [wordpress]
---
# MemberPress Form Customization

To alter the settings of a plugin in WordPress, we need to write PHP scripts with the *Hooks* set by the plugin developers. In this post, I will introduce some hooks from MemberPress and my examples of using them.

*To get the source code of MemberPress, please check [here](https://github.com/wp-premium/memberpress-basic)*

## MemberPress Common Hooks

#### Registration Form
>Form name: *'mepr_process_signup_form'*

| Hooks      | Description | usage |
| :----    |    :----   | :---- |
| mepr-checkout-after-email-field | after the email field in the default signup form | add custom field manually |
| mepr-checkout-before-submit | before the submit button on signup form | add reCaptcha; add words/fields|
| mepr-validate-signup     | user signup validation process| change error message |
| mepr-signup| signup information passing to the database | get username (not tested) |
| mepr-auto-login | Auto-login process after user passed signup validation | disable the auto-login |

#### Login Form
>Form name: *'mepr_loginform'*

| **Hooks**      | **Description** | **usage** |
| :----     |    :----   | :---- |
| mepr-login-form-before-submit | before the submit button on the login form | add reCaptcha; add words/fields|
| mepr-validate-login | user login validation process | change error message |
| mepr-login-redirect-url | url redirect to after user login | change redirect location manually |
| mepr-logout-url | url redirect to after user logout | change redirect location manually |

#### Forgot Password Form
>Form name: *'mepr_forgot_password_form'*

| **Hooks**      | **Description** | **usage** |
| :----    |    :----  | :---- |
| mepr-forgot-password-form|before the submit button on the forgot password form | add reCaptcha; add words/fields|
| mepr-validate-forgot-password | validation process of user information to request reset password | change error message |


More Hooks to come....

## Examples with Hooks

### Add reCaptcha to forms (with [BestWestSoft ReCaptcha](https://nl.wordpress.org/plugins/google-captcha/) plugin)

There are two ways to add reCaptcha to MemberPress forms. The codes can be added by using any snippet plugin or can be added to *function.php* in child theme. 

#### Method 1: use shortcode

The following code is to embed the reCaptcha to *Forgot Password Form*:
```php
<?php
function display_recaptcha_mepr_password_reset() {
 ?>
	<?php echo do_shortcode("[bws_google_captcha]"); ?>
<?php
}
add_action('mepr-forgot-password-form', 'display_recaptcha_mepr_password_reset');
```

-*<span style="color:#40867e;">'mepr-forgot-password-form'</span>*: the hook to allow developers to add content before the submit button. Feel free to change to hook, which is indicating the space before the submit button, from other forms to embed the reCaptcha to the forms.

Now you should see the reCaptcha show up on your form:

<img src="https://user-images.githubusercontent.com/41762593/84104965-eb16d080-a9e4-11ea-889a-a59bb7521e97.png" width="80%">

#### Method 2: Add enable/disable reCaptcha functionalities for customized forms in the plugin setting

For this method, please use [this article](https://support.bestwebsoft.com/hc/en-us/articles/202352499-How-to-add-reCaptcha-plugin-to-a-custom-form-on-my-WordPress-website-) as reference. 

Let's add reCaptcha to the MemberPress signup form.

<ins>**Step 1:** </ins><br>
First, we need to add the connection between the form and the reCaptcha plugin

```php
<?php
function add_custom_recaptcha_forms( $forms ) {
$forms['mepr_process_signup_form'] = array( "form_name" => "MemberPress General Sign Up Form" );

return $forms;
}
add_filter( 'gglcptch_add_custom_form', 'add_custom_recaptcha_forms' );
```

-*<span style="color:#40867e;">'mepr_process_signup_form'</span>*: the form name of the signup form in MemberPress. Feel free to change it to the name of the form you would like to add.<br>
-*<span style="color:#40867e;">"MemberPress General Sign Up Form"</span>*: the form name going to display in the reCaptcha setting, please see the image below.<br>
-*<span style="color:#40867e;">'gglcptch_add_custom_form'</span>*: the hook for BWS reCaptcha plugin to add custom forms to the *Setting* page.<br>

After adding the code above, we should see the form show up in the plugin setting page under general tap:

<img src="https://user-images.githubusercontent.com/41762593/84105012-0d105300-a9e5-11ea-85f0-22669d9e416f.png" width="80%" >

Now you can enable/disable the reCaptcha for the form from the plugin setting.

<ins>**Step 2:** </ins><br>
Now Add the plugin to the signup form:

```php
<?php
function display_recaptcha_mepr_signup() {
 ?>
	<?php echo apply_filters( 'gglcptch_display_recaptcha', '', 'mepr_process_signup_form' ); ?>	
<?php
}
add_action('mepr-checkout-before-submit', 'display_recaptcha_mepr_signup');
```

-*<span style="color:#40867e;">'mepr_process_signup_form'</span>*: Make sure the form name is the same as *Step 1*.<br>
-*<span style="color:#40867e;">'mepr-checkout-before-submit'</span>*: the hook in the signup form indicating the space before the submit button.<br>
-*<span style="color:#40867e;">'gglcptch_display_recaptcha'</span>*: hook in the reCaptcha plugin to show the reCaptcha in the corresponding form.<br>

<ins>**Step 3:** </ins><br>
We also need to add the validation process to make the reCaptcha effective:

```php
<?php
function validate_recaptcha_mepr_signup($errors) {
  $is_valid = apply_filters('gglcptch_verify_recaptcha', true,'mepr_process_signup_form');
  if(!$is_valid) {
    $errors[] = "Invalid Captcha value";
  }
  return $errors;
}
add_filter('mepr-validate-signup', 'validate_recaptcha_mepr_signup');
```

-*<span style="color:#40867e;">'gglcptch_verify_recaptcha'</span>*: the hook in the reCaptcha plugin to return the response from the user.<br>
-*<span style="color:#40867e;">'mepr-validate-signup'</span>*: the hook of the validation process for the MemberPress Signup Form.<br>

#### Add Error Messages for reCaptcha

After adding the reCaptcha, we need to create the correct error message in case the user forget to click the reCaptcha.

Here is the code to add error message for the *Forgot Password Form*
```php
<?php
function validate_recaptcha_mepr_password_reset($errors) {
	$is_valid = apply_filters('gglcptch_verify_recaptcha', true, 'string', 'mepr-forgot-password-form');
  if(!$is_valid) {
    $errors[] = "Invalid Captcha value";
  }
  return $errors;
}
add_filter('mepr-validate-forgot-password', 'validate_recaptcha_mepr_password_reset');

```
-*<span style="color:#40867e;">'gglcptch_verify_recaptcha'</span>*: the hook in the reCaptcha plugin to return the response from the user.<br>
-*<span style="color:#40867e;">'mepr-validate-forgot-password'</span>*: the validation process of the MemberPress Forgot Password Form.<br>

The error messages can be changed in<code>$errors[] =""</code>


-------------------------------------------------------------

more content to come.....

### Relocate the customized fields for Memberpress forms
