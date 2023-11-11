
A WooCommerce alapértelmezetten 6 típusú egyedi mezőt támogat úgynevezett built in módon. 

* Text
* Textarea
* Number
* Checkbox
* Radio
* Select (dropdown)

## A mezők létrehozásakor úgynevezett attributumok segítségével tudjuk meghatározni az egyes tulajdonságokat.

**Példa:**
```
add_action( 'woocommerce_product_options_general_product_data', 'add_custom_text_field' );
function add_custom_text_field() {
    woocommerce_wp_text_input( array(
        'id'          => '_custom_product_text_field', 
        'label'       => __('Custom Text Field', 'woocommerce'), 
        'placeholder' => __('Enter text here', 'woocommerce'),
        'desc_tip'    => true, 
        'description' => __('This is a custom text field.', 'woocommerce'), 
        'type'        => 'text',
        'default'     => '', 
    ));
}
```
A példa kód egy egyszerű text meta mezőt fog létrehozni, amely közvetlenül a **Álatalános** tabfülön az árak alatt jelenik meg. A poziciót, és hogy melyik tabfülbe szeretnénk elhelyezni, vagy esetleg egy teljesen újban szintén lehetséges.

* ID             -> Ez lesz a meta azonosítója, nem tartalmazhat speciális karaktereket vagy ékezetet. Minden meződnek eltérő azonosítóval kell rendelkeznie.
* LABEL          -> Ez lesz a mező címkéje, neve ami megjelenik az egyedi mező felett.
* PLACEHOLDER    -> Másnéven helykitöltő, egyes mezők esetében ez jelenik meg az input-ban. (például a text, vagy textarea esetében)
* DESC_TIP       -> A WooCommerce beépített Tooltip kezelője, ha az érték true, azaz igaz, a *description* azaz leírás tooltipként fog megjelenni.
* DESCRIPTION    -> A mező leírása, ami a mező alatt jelenik meg, vagy tooltip *true* esetében a tooltipben.
* TYPE           -> Itt határozod meg a mező típusát.
* DEFAULT        -> Legyen-e alapértelmezett érték megadva, ha nem hagy '' üresen.

Ha kész az egyedi mezőnk, gondoskodnunk kell arról, hogy ezt megfelelően kezeljük, és mentsük az adatbázisban

```
add_action( 'woocommerce_process_product_meta', 'save_custom_text_field' );
function save_custom_text_field( $post_id ) {
    if ( isset( $_POST['_custom_product_text_field'] ) ) {
        $custom_field_value = sanitize_text_field( $_POST['_custom_product_text_field'] );
        
        // Ha a mező üres, töröljük a metaadatot
        if ( empty( $custom_field_value ) ) {
            delete_post_meta( $post_id, '_custom_product_text_field' );
        } else {
            // Ha van érték, frissítjük a metaadatot
            update_post_meta( $post_id, '_custom_product_text_field', $custom_field_value );
        }
    } else {
        // Ha a mező nem létezik, töröljük a metaadatot
        delete_post_meta( $post_id, '_custom_product_text_field' );
    }
}
```

## Példa az összes típusú meta mezőre:

### Textarea típusú meta mező

```
add_action( 'woocommerce_product_options_general_product_data', 'add_custom_textarea_field' );
function add_custom_textarea_field() {
    woocommerce_wp_textarea_input( array(
        'id'          => '_custom_product_textarea',
        'label'       => __('Custom Textarea', 'woocommerce'),
        'placeholder' => __('Enter details here', 'woocommerce'),
        'description' => __('This is a custom textarea field.', 'woocommerce'),
    ));
}
```


### Number (szám) típusú meta mező

```
add_action( 'woocommerce_product_options_general_product_data', 'add_custom_number_field' );
function add_custom_number_field() {
    woocommerce_wp_text_input( array(
        'id'          => '_custom_product_number_field',
        'label'       => __('Custom Number Field', 'woocommerce'),
        'placeholder' => __('Enter a number', 'woocommerce'),
        'description' => __('This is a custom number field.', 'woocommerce'),
        'type'        => 'number',
        'custom_attributes' => array(
            'step'  => 'any',
            'min'   => '0'
        )
    ));
}
```

### Checkbox (jelölőnégyzet) típusú meta mező

```
add_action( 'woocommerce_product_options_general_product_data', 'add_custom_checkbox_field' );
function add_custom_checkbox_field() {
    woocommerce_wp_checkbox( array(
        'id'          => '_custom_product_checkbox',
        'label'       => __('Custom Checkbox', 'woocommerce'),
        'description' => __('Check this box for custom option.', 'woocommerce')
    ));
}
```

### Select (dropdown) típusú meta mező

```
add_action( 'woocommerce_product_options_general_product_data', 'add_custom_select_field' );
function add_custom_select_field() {
    woocommerce_wp_select( array(
        'id'          => '_custom_product_select',
        'label'       => __('Custom Select', 'woocommerce'),
        'options'     => array(
            ''        => __('Please select', 'woocommerce'),
            'option1' => __('Option 1', 'woocommerce'),
            'option2' => __('Option 2', 'woocommerce')
        )
    ));
}
```

### Radio (Rádió gombok) típusú meta mező

```
add_action( 'woocommerce_product_options_general_product_data', 'add_custom_radio_buttons' );
function add_custom_radio_buttons() {
    echo '<div class="options_group">';
    echo '<p class="form-field _custom_product_radio_field">';
    echo '<label>' . __('Custom Radio Buttons', 'woocommerce') . '</label>';
    echo '<input type="radio" name="_custom_product_radio" value="option1"> ' . __('Option 1', 'woocommerce') . '<br>';
    echo '<input type="radio" name="_custom_product_radio" value="option2"> ' . __('Option 2', 'woocommerce');
    echo '</p>';
    echo '</div>';
}
```
