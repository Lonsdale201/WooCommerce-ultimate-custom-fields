## Text mező létrehozása, mentése, frontend megjelenítése

```
add_action('woocommerce_product_options_general_product_data', 'custom_product_text_field');
function custom_product_text_field() {
    woocommerce_wp_text_input(
        array(
            'id' => '_custom_text_field',
            'label' => __('Egyedi szövegmező', 'woocommerce'), 
            'placeholder' => __('Írd be az egyedi szöveget', 'woocommerce'), 
            'desc_tip' => 'true',
            'description' => __('Adj meg egy egyedi értéket a termékhez.', 'woocommerce'), 
        )
    );
}

// Egyedi mező értékének mentése
add_action('woocommerce_process_product_meta', 'save_custom_product_text_field');

function save_custom_product_text_field($post_id) {
    $custom_field_value = isset($_POST['_custom_text_field']) ? sanitize_text_field($_POST['_custom_text_field']) : '';
    if (!empty($custom_field_value)) {
        update_post_meta($post_id, '_custom_text_field', esc_attr($custom_field_value));
    } else {
        delete_post_meta($post_id, '_custom_text_field'); 
    }
}

// Egyedi mező megjelenítése a termékoldalon
add_action('woocommerce_before_add_to_cart_button', 'display_custom_product_text_field');

function display_custom_product_text_field() {
    global $post;

    // Az egyedi mező értékének lekérése
    $custom_field_value = get_post_meta($post->ID, '_custom_text_field', true);

    // Ha van érték, megjelenítjük
    if (!empty($custom_field_value)) {
        echo '<p class="custom-field-text">' . esc_html($custom_field_value) . '</p>';
    }
}
```
