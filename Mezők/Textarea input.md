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

## Textarea mező létrehozása, mentése, frontend megjelenítése - támogatja bizonyos html elemek alkalmazását is
Ez a változat tartalmaz néhány html elemet, amit beírhatnak a textarea mezőbe. A fenti példa ezeket kiszűri, és plain menti el, ez viszont egy rugalmasabb szabályt alkalmaz.

```
add_action('woocommerce_product_options_general_product_data', 'custom_product_textarea_field');
function custom_product_textarea_field() {
    woocommerce_wp_textarea_input(
        array(
            'id' => '_custom_textarea_field', // Egyedi mező azonosítója
            'label' => __('Egyedi HTML textarea mező', 'woocommerce'), 
            'placeholder' => __('Írd be az egyedi HTML kódot', 'woocommerce'),
            'desc_tip' => 'true', 
            'description' => __('Adj meg egy egyedi értéket HTML formátumban a termékhez.', 'woocommerce'), 
        )
    );
}

// Egyedi textarea mező értékének mentése HTML-lel
add_action('woocommerce_process_product_meta', 'save_custom_textarea_field');

function save_custom_textarea_field($post_id) {
    $allowed_html = array(
        'a' => array(
            'href' => array(),
            'title' => array()
        ),
        'br' => array(),
        'em' => array(),
        'strong' => array(),
        'p' => array(),
        'ul' => array(),
        'ol' => array(),
        'li' => array(),
        'span' => array(
            'style' => array(),
            'class' => array()
        ),
        'div' => array(
            'class' => array(),
            'style' => array()
        ),
        'h1' => array(),
        'h2' => array(),
        'h3' => array(),
        'h4' => array(),
        'h5' => array(),
        'h6' => array(),
    );

    $custom_field_value = isset($_POST['_custom_textarea_field']) ? wp_kses($_POST['_custom_textarea_field'], $allowed_html) : '';
    
    if (!empty($custom_field_value)) {
        update_post_meta($post_id, '_custom_textarea_field', $custom_field_value);
    } else {
        delete_post_meta($post_id, '_custom_textarea_field'); 
    }
}

// Egyedi textarea mező HTML formázással történő megjelenítése a termékoldalon
add_action('woocommerce_before_add_to_cart_button', 'display_custom_textarea_field');

function display_custom_textarea_field() {
    global $post;

    // Az egyedi textarea mező értékének lekérése
    $custom_field_value = get_post_meta($post->ID, '_custom_textarea_field', true);

    if (!empty($custom_field_value)) {
        echo '<div class="custom-field-textarea">' . wp_kses_post($custom_field_value) . '</div>';
    }
}
```

## Textarea mező létrehozása, mentése, speciális frontend megjelenítés
Ez a változat szintén tisztítja a bementelt, azaz a html nem engedélyezett, azonban ha a textarea mezőbe beírt szövegekben van sortörés, akkor a frontend oldalon ennek megfelelően formzáva jelenítjük meg.

```
add_action('woocommerce_product_options_general_product_data', 'custom_product_textarea_field');
function custom_product_textarea_field() {
    woocommerce_wp_textarea_input(
        array(
            'id' => '_custom_textarea_field',
            'label' => __('Egyedi textarea mező', 'woocommerce'), 
            'placeholder' => __('Írd be az egyedi szöveget', 'woocommerce'), 
            'desc_tip' => 'true', 
            'description' => __('Adj meg egy hosszabb egyedi értéket a termékhez.', 'woocommerce'), 
        )
    );
}

add_action('woocommerce_process_product_meta', 'save_custom_textarea_field');
function save_custom_textarea_field($post_id) {
    $custom_field_value = isset($_POST['_custom_textarea_field']) ? sanitize_textarea_field($_POST['_custom_textarea_field']) : '';
    
    if (!empty($custom_field_value)) {
        update_post_meta($post_id, '_custom_textarea_field', esc_attr($custom_field_value));
    } else {
        delete_post_meta($post_id, '_custom_textarea_field'); 
    }
}

add_action('woocommerce_before_add_to_cart_button', 'display_custom_textarea_field');
function display_custom_textarea_field() {
    global $post;

    $custom_field_value = get_post_meta($post->ID, '_custom_textarea_field', true);

    if (!empty($custom_field_value)) {
        // A megjelenítendő szöveg sorokra tördelése
        $lines = explode("\n", esc_html($custom_field_value));
        echo '<div class="custom-field-textarea">';
        foreach ($lines as $line) {
            echo '<p>' . $line . '</p>';
        }
        echo '</div>';
    }
}
```
