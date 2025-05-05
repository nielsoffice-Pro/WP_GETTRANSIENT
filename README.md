# WP_GET_TRANSIENT
get_trnasient WP Function will pass the value from do_action in callback to another function or into shortcode WordPress PHP

<br>Retrieves the value of a transient
get_transient() is a WordPress PHP function that retrieves the value of a transient. The function takes a single parameter, which is the name of the transient you want to retrieve. It checks if the transient exists and if it does, it returns the value of the transient. If the transient does not exist or has expired, the function returns false. To save a transient, you should use the set_transient() function. 

 `do_action( 'some_custom_action', 10 )` by default if no specific discount data is passed. This means that, when the action is called, it will check if a value (like the role-based discount) exists; otherwise, it will use `10%` as the fallback.

Here’s how you can implement this:

---

### **1. Modify the Action to Accept Default Value**

You can adjust the action so it accepts a **default value** (like `10%`) and allows it to be overridden later (for specific roles or conditions).

### **Action Hook with Default Value**

```php
add_action( 'some_custom_action', 'set_discount_by_user_role', 10, 1 );

function set_discount_by_user_role( $discount = 10 ) {
    if ( ! is_user_logged_in() ) {
        return;
    }

    $user = wp_get_current_user();
    $roles = (array) $user->roles;

    // Override the default discount based on user role
    if ( in_array( 'client', $roles ) ) {
        $discount = 10;
    } elseif ( in_array( 'manager', $roles ) ) {
        $discount = 5;
    } elseif ( in_array( 'subscriber', $roles ) ) {
        $discount = 2;
    }

    // Save the discount using the user ID so it's user-specific
    set_transient( 'custom_discount_' . $user->ID, $discount, 60 ); // Expires in 60 seconds
}
```

### Explanation:
- **$discount = 10**: This sets a default value for the discount parameter. If no discount is passed into the action (from the call), it will fall back to 10%.
- If the user has a role (like `client`, `manager`, or `subscriber`), we adjust the discount accordingly.

---

### **2. Update the Call to `do_action()`**

Now, when you call the `do_action( 'some_custom_action' )`, you can pass in a specific discount if needed, or just rely on the default value of `10%`.

- **Without a custom value (uses default 10%)**:
    ```php
    do_action( 'some_custom_action' );  // Uses default 10% if no discount is passed
    ```

- **With a custom value (overrides the default)**:
    ```php
    do_action( 'some_custom_action', 5 );  // This sets discount to 5%
    ```

---

### **3. Shortcode to Display Discount**

The shortcode remains the same, as it will display the discount that was saved by `set_discount_by_user_role`:

```php
add_shortcode( 'user_discount', 'display_user_discount_shortcode' );

function display_user_discount_shortcode( $atts ) {
    if ( ! is_user_logged_in() ) {
        return 'Please log in to see your discount.';
    }

    $user_id = get_current_user_id();
    $discount = get_transient( 'custom_discount_' . $user_id );

    if ( $discount === false ) {
        return 'No discount available.';
    }

    return "Your discount is {$discount}%";
}
```

### **4. Use the Action and Shortcode**

To trigger this logic:

1. **Call the action in your template** (you can call it in the header, footer, or before rendering the page):

    ```php
    do_action( 'some_custom_action' );  // Default 10% discount is applied
    ```

2. **Call the shortcode on a page**:

    ```php
    echo do_shortcode('[user_discount]');
    ```

---

### **Handling Default Values Dynamically**
If you want the default value to be configurable (e.g., change the default discount percentage without editing the code), you could set a **constant or option** for the default discount:

```php
if ( ! defined( 'DEFAULT_DISCOUNT' ) ) {
    define( 'DEFAULT_DISCOUNT', 10 ); // Default value of 10%
}

add_action( 'some_custom_action', 'set_discount_by_user_role', 10, 1 );

function set_discount_by_user_role( $discount = DEFAULT_DISCOUNT ) {
    if ( ! is_user_logged_in() ) {
        return;
    }

    $user = wp_get_current_user();
    $roles = (array) $user->roles;

    // Override the default discount based on user role
    if ( in_array( 'client', $roles ) ) {
        $discount = 10;
    } elseif ( in_array( 'manager', $roles ) ) {
        $discount = 5;
    } elseif ( in_array( 'subscriber', $roles ) ) {
        $discount = 2;
    }

    // Save the discount using the user ID so it's user-specific
    set_transient( 'custom_discount_' . $user->ID, $discount, 60 ); // Expires in 60 seconds
}
```

Now, if you need to change the default percentage, you can just change the `DEFAULT_DISCOUNT` constant.

---

### **Summary**

- We’ve set up the action `do_action( 'some_custom_action', 10 )` with `10%` as the default value.
- When the action is triggered, it checks the user’s role and adjusts the discount accordingly.
- The shortcode will display the discount that was set during the action hook.
  
You can now trigger `do_action()` with different discount values based on specific conditions or leave it as the default `10%`. Does that solution work for you?
