## Textarea mező létrehozása, mentése, frontend megjelenítése

```
add_action('woocommerce_product_options_general_product_data', 'custom_product_textarea_field');
function custom_product_textarea_field() {
    woocommerce_wp_textarea_input(
        array(
            'id' => '_custom_textarea_field', // Egyedi mező azonosítója
            'label' => __('Egyedi textarea mező', 'woocommerce'), 
            'placeholder' => __('Írd be az egyedi szöveget', 'woocommerce'), 
            'desc_tip' => 'true', 
            'description' => __('Adj meg egy hosszabb egyedi értéket a termékhez.', 'woocommerce'), 
        )
    );
}

// Egyedi textarea mező értékének mentése
add_action('woocommerce_process_product_meta', 'save_custom_textarea_field');

function save_custom_textarea_field($post_id) {
    $custom_field_value = isset($_POST['_custom_textarea_field']) ? sanitize_textarea_field($_POST['_custom_textarea_field']) : '';
    
    if (!empty($custom_field_value)) {
        update_post_meta($post_id, '_custom_textarea_field', esc_attr($custom_field_value));
    } else {
        delete_post_meta($post_id, '_custom_textarea_field'); 
    }
}

// Egyedi textarea mező megjelenítése a termékoldalon
add_action('woocommerce_before_add_to_cart_button', 'display_custom_textarea_field');

function display_custom_textarea_field() {
    global $post;

    // Az egyedi textarea mező értékének lekérése
    $custom_field_value = get_post_meta($post->ID, '_custom_textarea_field', true);

    // Ha van érték, megjelenítjük
    if (!empty($custom_field_value)) {
        echo '<p class="custom-field-textarea">' . nl2br(esc_html($custom_field_value)) . '</p>';
    }
}

```
