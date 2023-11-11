
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
    global $post;

    // Lekérjük a mentett értéket
    $checkbox_value = get_post_meta( $post->ID, '_custom_product_checkbox', true );

    woocommerce_wp_checkbox( array(
        'id'          => '_custom_product_checkbox',
        'label'       => __('Custom Checkbox', 'woocommerce'),
        'description' => __('Check this box for custom option.', 'woocommerce'),
        'value'       => $checkbox_value,
        'cbvalue'     => 'yes',
        'checked'     => ( $checkbox_value === 'yes' ) ? 'checked' : ''
    ));
}
```

A Checkbox és a radio speciálisabb, mivel itt is kezelni kell a bejelölt értéket. Így a mentés a következő:

```
add_action( 'woocommerce_process_product_meta', 'save_custom_checkbox_field' );
function save_custom_checkbox_field( $post_id ) {
    $checkbox_value = isset( $_POST['_custom_product_checkbox'] ) ? 'yes' : 'no';
    update_post_meta( $post_id, '_custom_product_checkbox', $checkbox_value );
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
Mentés:

```
add_action( 'woocommerce_process_product_meta', 'save_custom_select_field' );
function save_custom_select_field( $post_id ) {
    // Ellenőrizzük, hogy a select mező értéke be van-e állítva
    if ( isset( $_POST['_custom_product_select'] ) ) {
        // Tisztítás
        $select_value = sanitize_text_field( $_POST['_custom_product_select'] );

        // Frissítsük a termék metaadatát a kiválasztott értékkel
        update_post_meta( $post_id, '_custom_product_select', $select_value );
    }
}
```

### Radio (Rádió gombok) típusú meta mező

```
add_action( 'woocommerce_product_options_general_product_data', 'add_custom_radio_buttons' );
function add_custom_radio_buttons() {
    global $post;

    // Lekérjük a mentett értéket - lokális változó
    $saved_value = get_post_meta( $post->ID, '_custom_product_radio', true );

    // Rádiógombok megjelenítése
    echo '<div class="options_group">';
    echo '<p class="form-field _custom_product_radio_field">';
    echo '<label>' . __('Custom Radio Buttons', 'woocommerce') . '</label>';
    echo '<input type="radio" name="_custom_product_radio" value="option1" ' . checked( $saved_value, 'option1', false ) . '> ' . __('Option 1', 'woocommerce') . '<br>';
    echo '<input type="radio" name="_custom_product_radio" value="option2" ' . checked( $saved_value, 'option2', false ) . '> ' . __('Option 2', 'woocommerce');
    echo '</p>';
    echo '</div>';
}
```

Mentés:

A Radiogombok esetében van némi extra, mivel vissza kell töltenünk a bejelölt értéket, ezért valamivel eltérőbb lesz a többinél az alapértelmezett kódja
```
function save_custom_radio_buttons( $post_id ) {
    if ( isset( $_POST['_custom_product_radio'] ) ) {
        $radio_value = sanitize_text_field( $_POST['_custom_product_radio'] );
        update_post_meta( $post_id, '_custom_product_radio', $radio_value );
    } else {
    }
}
add_action( 'woocommerce_process_product_meta', 'save_custom_radio_buttons' );
```


### Speciális Multiple select mező

Amennyiben olyan mezőt szeretnél készíteni, amiben egy vagy több értéket is ki lehet jelölni, úgy felhasználhatjuk a WooCommerce által is használt select2 kiegészítőt
Itt két érték van "value1 stb stb", és mellé egy "Label" A label paraméterek lesznek azok amik a mezőben láthatóak lesznek a felhasználó számára, de az adatbázisban a value1 stb értékek kerülnek mentésre.

```
add_action( 'woocommerce_product_options_general_product_data', 'add_custom_multiple_select_field' );
function add_custom_multiple_select_field() {
    global $post;

    // Lekérjük a mentett értékeket
    $saved_values = get_post_meta( $post->ID, '_custom_product_multiple_select', true );
    $saved_values = !empty($saved_values) ? explode(',', $saved_values) : array();

    // Opciók definiálása
    $options = array(
        'value1' => __('Label 1', 'woocommerce'),
        'value2' => __('Label 2', 'woocommerce'),
        'value3' => __('Label 3', 'woocommerce'),
        'value4' => __('Label 4', 'woocommerce')
    );

    echo '<p class="form-field">';
    echo '<label for="_custom_product_multiple_select">' . __('Custom Multiple Select', 'woocommerce') . '</label>';
    echo '<select id="_custom_product_multiple_select" name="_custom_product_multiple_select[]" class="wc-enhanced-select" multiple="multiple">';
    foreach ($options as $value => $label) {
        echo '<option value="' . esc_attr($value) . '" ' . (in_array($value, $saved_values) ? 'selected' : '') . '>' . esc_html($label) . '</option>';
    }
    echo '</select>';
    echo '</p>';
}
```

Mentés esetében, hogy ezek az értékek szűrő kompatibilisek legyenek az *implode* segítségével vesszőalapú elválasztási technikát alkalmazhatunk, így például ezeket az értékeket a JetSmartFilter képes lesz érézkelni, és szűrni:

```
add_action( 'woocommerce_process_product_meta', 'save_custom_multiple_select_field' );
function save_custom_multiple_select_field( $post_id ) {
    if ( isset( $_POST['_custom_product_multiple_select'] ) ) {
        $select_values = array_map('sanitize_text_field', $_POST['_custom_product_multiple_select']);
        $select_values = implode(',', $select_values); // Vesszővel elválasztott stringgé alakítjuk
        update_post_meta( $post_id, '_custom_product_multiple_select', $select_values );
    } else {
        delete_post_meta( $post_id, '_custom_product_multiple_select' );
    }
}
```

Fontos megjegyezni, hogy a frontend -en való kiíráskor alapértelmezetten a value azaz az adatbázisban tárolt értéket fogja megjeleníteni. Nézzünk meg egy shortcode példát az alapvető value értékek megjelenítésére, és egy label alapú megjelenítésre is.

value alapú alapértelmezett megjelenítése:
**Shortcode: [custom_multiple_select]**

```
function display_custom_multiple_select_values() {
    global $post;

    // Ellenőrizzük, hogy van-e érvényes termék ID
    if ( !is_a( $post, 'WP_Post' ) || get_post_type( $post->ID ) !== 'product' ) {
        return '';
    }

    // Lekérjük a mentett értékeket
    $saved_values = get_post_meta( $post->ID, '_custom_product_multiple_select', true );
    if ( !empty($saved_values) ) {
        // Vesszővel elválasztva jelenítjük meg az értékeket
        return esc_html( implode( ', ', explode(',', $saved_values) ) );
    }

    return __('Nincs beállított érték', 'woocommerce');
}
add_shortcode( 'custom_multiple_select', 'display_custom_multiple_select_values' );
```

Label alapú megjelenítése:
**Shortcode: [custom_multiple_select_labels]**

```
function display_custom_multiple_select_labels() {
    global $post;

    // Ellenőrizzük, hogy van-e érvényes termék ID
    if ( !is_a( $post, 'WP_Post' ) || get_post_type( $post->ID ) !== 'product' ) {
        return '';
    }

    // Lekérjük a mentett értékeket
    $saved_values = get_post_meta( $post->ID, '_custom_product_multiple_select', true );
    if ( !empty($saved_values) ) {
        $saved_values = explode(',', $saved_values);

        // Opciók definiálása (ugyanazok, mint a mező létrehozásakor)
        $options = array(
            'value1' => __('Label 1', 'woocommerce'),
            'value2' => __('Label 2', 'woocommerce'),
            'value3' => __('Label 3', 'woocommerce'),
            'value4' => __('Label 4', 'woocommerce')
        );

        // Az értékek átkonvertálása label-ekre
        $labels = array();
        foreach ($saved_values as $value) {
            if (isset($options[$value])) {
                $labels[] = $options[$value];
            }
        }

        // Vesszővel elválasztva jelenítjük meg a label-eket
        return implode(', ', $labels);
    }

    return __('Nincs beállított érték', 'woocommerce');
}
add_shortcode( 'custom_multiple_select_labels', 'display_custom_multiple_select_labels' );
```
