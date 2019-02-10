---
layout: post
title: "Adding keyboard shortcut to a web page"
---

Step by step approach to add keyboard short to a web page using Javascript/Jquery.

<!-- more -->

Nowadays everyone have either mouse or touch pad or similar to move around the web page. But, having keyboard shortcut is still a great ingredient on a web page for users. Especially for the regular users of a website who knows exactly which page has what. In this article I would like to explain how to achieve this. I have used Jquery and  basic Javascript code to do it.

#### Keydown Event
First we need to catch the keydown event on the page, as shown below:
```javascript
$(window).keydown(function(event) {
    // code goes here…..
});
```
where, event object contains the key details.

#### event.keyCode
"event.which" property normalizes event.keyCode and event.charCode. It is recommended to watch event.which for keyboard key input. event.which also normalizes button presses (mousedown and mouseupevents), reporting 1 for left button, 2 for middle, and 3 for right.
`event.ctrlKey`
"`event.ctrlKey`" tells us whether user has pressed "Ctrl" key or not. This function returns a boolean value (true/false).
Cool, we are now good to go to write logic to handle the events.

##### Step 1: Include Jquery
```javascript
<script src="https://code.jquery.com/jquery-3.3.1.js" ></script>
```
For more information about the other CDNs or if you would like to host the Jquery library locally in your project, then please visit Jquery site.
##### Step 2: Create element
```html
<input type="button" name="btn1" class="btn1" value="Button" />
```
##### Step 3: Click event
Add the below code which shows an alert on click of the button.
```javascript
$('.btn1').click(function(e) {
    alert("Hey, you clicked the button!");
});
```
##### Step 4: Handle key press
As discussed above, Jquery's "keydown" method helps us handling the user's key presses. In this example, I would like to trigger the button click event when user presses F2 key.
```javascript
if(event.which == 113) { 
   $('.btn1').click();
   event.preventDefault();
   return;
}
```
where, "113" is the key code for the key F2 on the keyboard. When a user press F2 key, we need to simply trigger the click event on the button, which we have already done in Step 3.
Whoop whoop, that's everything we need to do :)
#### Full Working Example:
Please copy & paste the below fully-working html code.
```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <title>Jquery Shortcut Key</title>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
        <script src="https://code.jquery.com/jquery-3.3.1.js"></script>
    </head>
    <body>
        <label>Button (F2): </label><input type="button" name="btn1" class="btn1" value="Button" /> <br />
        <label>Link (^F2): </label><a href="#" class="link1">Link</a> <br />
        <label>Text box (^Y): </label><input type="text" name="text1" class="text1" placeholder="some value" /> <br />
        
        <script>
            $(document).ready(function(){
                $(window).keydown(function(event) {
                    if(!event.ctrlKey && event.which == 113) { 
                        $('.btn1').click();
                        event.preventDefault();
                        return;
                    }
                    if(event.ctrlKey && event.which == 113) {
                        $('.link1').click();
                        event.preventDefault();
                        return;
                    }
                    if(event.ctrlKey && event.which == 89) {
                        $('.text1').focus();
                        event.preventDefault();
                        return;
                    }
                });
                $('.btn1').click(function(e) {
                    alert("Hey, you clicked the button!");
                });
                $('.link1').click(function(e) {
                    alert("Hey, you clicked the link!");
                });
            });
        </script>
    </body>
</html>
```