## Checkbox mező létrehozása, mentése, frontend megjelenítése
A checkbox szöveget jelenít meg, de csak akkor, he bekapcsolták. (frontend oldalon) A Checkbox két értéket ment: yes és no

```
add_action('woocommerce_product_options_general_product_data', 'custom_product_checkbox_field');
function custom_product_checkbox_field() {
    woocommerce_wp_checkbox(
        array(
            'id' => '_custom_checkbox_field',
            'label' => __('Egyedi Checkbox mező', 'woocommerce'), 
            'description' => __('Jelöld be.', 'woocommerce')
        )
    );
}

add_action('woocommerce_process_product_meta', 'save_custom_checkbox_field');
function save_custom_checkbox_field($post_id) {
    $custom_field_value = isset($_POST['_custom_checkbox_field']) ? 'yes' : 'no'; 
    
    update_post_meta($post_id, '_custom_checkbox_field', $custom_field_value);
}

add_action('woocommerce_before_add_to_cart_button', 'display_custom_checkbox_field');
function display_custom_checkbox_field() {
    global $post;

    $custom_field_value = get_post_meta($post->ID, '_custom_checkbox_field', true);

    // Csak akkor jelenítjük meg, ha a checkbox be van jelölve
    if ($custom_field_value === 'yes') {
        echo '<p class="custom-field-checkbox">Az egyedi funkció aktív.</p>';
    }
}
```
