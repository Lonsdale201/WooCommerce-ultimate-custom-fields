## Number mező létrehozása, mentése, frontend megjelenítése
Használd a *floatval()* mentésnél, ha tizedest szeretnél használni, vagy *intval()* ami egész számként kezeli
```

add_action('woocommerce_product_options_general_product_data', 'custom_product_number_field');
function custom_product_number_field() {
    woocommerce_wp_text_input(
        array(
            'id' => '_custom_number_field',
            'label' => __('Egyedi szám mező', 'woocommerce'), 
            'placeholder' => __('Írd be az egyedi számot', 'woocommerce'), 
            'desc_tip' => 'true', 
            'description' => __('Adj meg egy egyedi számértéket a termékhez.', 'woocommerce'), 
            'type' => 'number', 
            'custom_attributes' => array(
                'min' => '1', // Min érték
                'max' => '100', // Max  érték
                'step' => '1' // léptetés
            )
        )
    );
}

add_action('woocommerce_process_product_meta', 'save_custom_number_field');
function save_custom_number_field($post_id) {
    $custom_field_value = isset($_POST['_custom_number_field']) ? floatval($_POST['_custom_number_field']) : '';

    if (!empty($custom_field_value)) {
        update_post_meta($post_id, '_custom_number_field', $custom_field_value);
    } else {
        delete_post_meta($post_id, '_custom_number_field'); 
    }
}

add_action('woocommerce_before_add_to_cart_button', 'display_custom_number_field');
function display_custom_number_field() {
    global $post;

    $custom_field_value = get_post_meta($post->ID, '_custom_number_field', true);

    if (!empty($custom_field_value)) {
        echo '<p class="custom-field-number">' . esc_html($custom_field_value) . '</p>';
    }
}
```
